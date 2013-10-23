[Source Code](https://github.com/sakaiproject/Hilary/blob/master/node_modules/oae-content/lib/api.js)

# Library

## Requirements

Every user and group (principal) has a library of content that has been shared with them. Whenever a piece of content is shared with a user, they are given access permissions to that content, AND it is stored in their library. When viewing a user's library, the items that will be displayed are different based on the status of the content:

<table>
  <thead>
    <tr>
      <th>
      <th>Anonymous User
      <th>Logged in user
      <th>Library Owner
      <th>Group Owner
  <tbody>
    <tr>
      <th>Public Visibility
      <td>Visible
      <td>Visible
      <td>Visible
      <td>Visible
    <tr>
      <th>Logged in Visibility
      <td>Hidden
      <td>Visible
      <td>Visible
      <td>Visible
    <tr>
      <th>Private Visibility
      <td>Hidden
      <td>Hidden
      <td>Visible
      <td>Visible
</table>

Note that any private content will not show up in another user's library, even if the content has been shared with the currently authenticated user. This is currently a performance consideration which may be addressed at a later time.

In addition to the above requirements, the library should be sorted by "Last Modified", ascending. Currently, there are no other sorting mechanisms required, but we anticipate there being more sorting requirements in the future.

## Data Model

The goal is to provide efficient access to a paged representation of a user's library from Cassandra, as it will be accessed quite frequently. To do this, we have come up with the following schema:

<table>
  <thead>
    <tr>
      <th>Column Family: LibraryByPrincipal
  <tbody>
    <tr>
      <th>u:cam:nicolaas#PUBLIC
      <td>1348067316:c:cam:License.txt
      <td>1348067316c:cam:ForEveryone.xls
    <tr>
      <th>u:cam:nicolaas#LOGGEDIN
      <td>1348067316:c:cam:License.txt
      <td>1348067316:c:cam:ForEveryone.xls
      <td>1348065000:c:cam:OnlyLoggedIn.txt
    <tr>
      <th>u:cam:nicolaas
      <td>1448065000:c:cam:SuperSecretDocument.txt
      <td>1348067316:c:cam:License.txt
      <td>1348067316:c:cam:ForEveryone.xls
      <td>1348065000:c:cam:OnlyLoggedIn.txt
</table>
      
In this schema, Cassandra will automatically have the content items sorted by last modified. Using column slices, we will be able to "page" through the data as well. By separating PUBLIC, LOGGEDIN, and PRIVATE content, we're able to perform all of the above in all content visibility use-cases.

Writing content involves some denormalization. Here is what is involved in updating the libraries:

**1. Content is shared with a user** - Place the content item, with its lastModified date into the appropriate visibility buckets of the user.

```sql
INSERT INTO ContentMembers VALUES (contentId="c:cam:License.txt", "u:cam:nicolaas"="true");
INSERT INTO LibraryByPrincipal VALUES (principalId="u:cam:nicolaas", "1348067316:c:cam:License.txt"="true");
INSERT INTO LibraryByPrincipal VALUES (principalId="u:cam:nicolaas:LOGGEDIN", "1348067316:c:cam:License.txt"="true");
INSERT INTO LibraryByPrincipal VALUES (principalId="u:cam:nicolaas:PUBLIC", "1348067316:c:cam:License.txt"="true");
```

**2. Content is removed from a library** - Remove the content item, given its lastModified date, from all appropriate visibility buckets of the user. (e.g., "delete $contentLastModified:c:cam:License.txt" from LibraryByPrincipal where principalId = 'u:cam:nicolaas')

```sql
DELETE "u:cam:nicolaas" FROM ContentMembers WHERE contentId = "c:cam:License.txt";
DELETE "1348067316:c:cam:License.txt" FROM LibraryByPrincipal WHERE principalId="u:cam:nicolaas";
DELETE "1348067316:c:cam:License.txt" FROM LibraryByPrincipal WHERE principalId="u:cam:nicolaas:LOGGEDIN";
DELETE "1348067316:c:cam:License.txt" FROM LibraryByPrincipal WHERE principalId="u:cam:nicolaas:PUBLIC";
```

**3. Content is updated** - Replace the content item attached to the previous timestamp with a content item hooked to the new timestamp. This is done for all users who have the content item in their library

```sql
-- Get all users who have this in their library
SELECT * FROM ContentMembers WHERE contentId="c:cam:License.txt"
 
-- For each of those user libraries, do this:
DELETE "1348067316:c:cam:License.txt" FROM LibraryByPrincipal WHERE principalId="u:cam:nicolaas";
DELETE "1348067316:c:cam:License.txt" FROM LibraryByPrincipal WHERE principalId="u:cam:nicolaas:LOGGEDIN";
DELETE "1348067316:c:cam:License.txt" FROM LibraryByPrincipal WHERE principalId="u:cam:nicolaas:PUBLIC";
INSERT INTO LibraryByPrincipal VALUES (principalId="u:cam:nicolaas", "1348070000:c:cam:License.txt"="true")
INSERT INTO LibraryByPrincipal VALUES (principalId="u:cam:nicolaas:LOGGEDIN", "1348070000:c:cam:License.txt"="true")
INSERT INTO LibraryByPrincipal VALUES (principalId="u:cam:nicolaas:PUBLIC", "1348070000:c:cam:License.txt"="true")
```

This technique is thought to be the most performant way to manage library updates and time-based sorting, but has some draw-backs

**Known Drawbacks:**

* If there is a failure between the time that the content is updated and all libraries are updated, there is now no way to recover without doing a full scan of all affected user libraries and plucking out the stale entry and updating the content item.
    * Potential solution is that we could record data about the failure: e.g., the lastModified of the content item before it was updated, and the users in the library, then retry the queries.

* The delete and insert queries are done using a single BATCH Cassandra query. However, as delete and insert happen at the same time, it is possible that a race condition forms that can result in duplicate entries in libraries when content is updated concurrently: 

    * Content with timestamp 1 is updated at the same time, both update requests (Update A and Update B) read oldLastModified of 1
    * Update A persists content with newLastModified = 2
    * Update B persists content with newLastModified = 3
    * Update A Deletes library entries with oldLastModified = 1
    * Update A Inserts library entries with newLastModified = 2
    * Update B Deletes library entries with oldLastModified = 1
    * Update B Inserts library entries with newLastModified = 3
    * **Result:** Duplicate entries in the libraries: 3:c:cam:License.txt and 2:c:cam:License.txt

        * **Potential solution:** By providing the ability to regenerate the LibraryByPrincipal CF on-the-fly if it doesn't exist, we can provide an easy way for support staff to safely recover a user's library: just delete it from the LibraryByPrincipal CF
        * **Potential solution:** Assuming duplicates will generally occur within the same "page" of paged access, detect duplicates on the fly and remove them. This will at least remove same-page duplicates.
