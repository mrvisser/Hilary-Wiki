The Authz API encompasses the logic required for ACLs, Roles, and Permissions for private content. It also takes on the notion of Group Membership, since permissions are propagated down a chain of group membership.

**Source Code:** https://github.com/sakaiproject/Hilary/blob/master/node_modules/oae-authz/lib/api.js

# Roles

## Requirements

* A role can be assigned to a principal on a resource instance
* A role can be removed from a principal on a resource instance
* A role can be updated from a principal on a resource instance
* A role is an arbitrary string, defined by the consumer of the Authz API
* It should be possible to determine if a user has a particular role on a resource instance (**hasRole**)
* It should be possible to determine what role a user has on a resource instance (**getRole**)
* It should be possible to determine, given a type of resource (e.g., content, group...), what resource instances the user has any role (**getRolesForResourceType**)
    * **Use Case:** Get me all the content to which I have access
    * **Use Case:** Get me all the groups of which I am a member
* It should be possible to determine, given a type of resource (e.g., content, group...) and an array of principals, what resource instants all principals have on that resource type (**getRolesForPrincipalsAndResourceType**)
    * **Use Case:** Get me all the groups of which my parent groups are a member
* A role cannot be assigned to a principal on a group resource instance. The group resource type is reserved to be interacted with solely through group membership functions (see Group Membership requirements below)
 
## Data Model

### Column Family: Roles

A dynamic column family that enumerates role association between a principal (row key; e.g., user, group) and some resource (column name; e.g., content, group).

**Row Key:** The row key is the principal UUID, which is scoped by "principal type", "tenant alias" and "principal id", respectively. Therefore, a principal UUID of: "u:cam:mrvisser" would represent a user from Cambridge with userid 'mrvisser'.

**Column Name:** The column name is the resource UUID, which is scoped by "resource type", "tenant alias" and "resource id", respectively. Therefore, a resource UUID of "c:cam:Foo.docx" represents a content item from Cambridge with resource id "Foo.docx"

**Column Values:** The value of each column is the principal's role on the resource.

<table>
  <thead>
    <tr>
      <th>Row Key (principal)
  <tbody>
    <tr>
      <th>u:cam:mrvisser
      <th>c:cam:Foo.docx
      <th>c:gat:Instructions.txt
      <th>g:cam:cheese-lovers
      <th>g:cam:my-group
      <th>g:gat:georgia-tech-global-network
    <tr>
      <td>
      <td>manager
      <td>viewer
      <td>member
      <td>administrator
      <td>member
    <tr>
      <th>u:cam:simong
      <th>c:cam:Foo.docx
      <th>g:cam: pizza-lovers
    <tr>
      <td>
      <td>viewer
      <td>member
    <tr>
      <th>g:cam:cheese-lovers
      <th>g:cam: pizza-lovers
      <th>c:gat:some-content
    <tr>
      <td>
      <td>member
      <td>viewer
</table>

<table>
  <thead>
    <tr>
      <th>Use Case
      <th>Query
  <tbody>
    <tr>
      <td>Given a principal, give me all of their group memberships (paged by 2)
      <td>`select first 2 'g:' .. '' from Roles where principal = ?`
    <tr>
      <td>
      <td>`select first 2 'g:gat:georgia-tech-global-network' .. '' from Roles where principal = ?`
    <tr>
      <td>Given a principal, give me all the content they have shared with them (paged by 2)
      <td>`select first 2 'c:' .. '' from Roles where principal = ?`
    <tr>
      <td>Given a principal, give me all their group memberships for their tenant (paged by 2)
      <td>`select first 2 'g:cam:' .. '' from Roles where principal = ?`
    <tr>
      <td>Given a principal, get their entire group membership ancestry
      <td>
```javascript
var memberships = "select 'g:' .. 'g:|' from Roles where principal = ?"
var indirectMemberships = "select 'g:' .. 'g:|' from Roles where principal in $memberships"
 
var newMemberships = indirectMemberships - memberships;
memberships.addAll(indirectMemberships);
while (!newMemberships.isEmpty()) {
    indirectMemberships = "select 'g:' .. 'g:|' from Roles where principal in $newMemberships"
    newMemberships = indirectMemberships - memberships;
    memberships.addAll(indirectMemberships);
}
 
return memberships;
```
</table>

# Group Memberships

## Requirements

* A principal can be added to a group, with a particular role in that group
* A principal can be removed from a group
* A principals role in a group can be updated
* A group role has the same restrictions as a regular role: it can be any arbitrary string defined by the consumer of the group membership
* It should be possible to determine all the direct members of a group
* It should be possible to determine, given a principal, all the groups of which they are a member, either directly or VIA indirect membership (e.g., member of a group, which is a member of a group, etc...)

## Data Model

### Column Family: GroupMembers

A dynamic column family that enumerates the principals that are a member of a group.

**Row Key:** The UUID of the group whose membership follows (e.g., g:cam:group-id)

**Column Names:** The column names are the principal UUIDs that are a member of the group in some capacity (e.g., u:cam:mrvisser)

**Column Values:** The value of each column is the role that the principal has in the group

<table>
  <thead>
    <tr>
      <th>Row Key (group)
  <tbody>
    <tr>
      <th>g:oae:oae-team
      <th>g:oae:oae-backend
      <th>g:oae:oae-frontend
      <th>u:oae:anthony
    <tr>
      <td>
      <td>member
      <td>member
      <td>manager
    <tr>
      <th>g:oae:oae-backend
      <th>u:oae:mrvisser
      <th>u:oae:simong
      <th>u:gat:stuartf
    <tr>
      <td>
      <td>member
      <td>member
      <td>member
    <tr>
      <th>g:oae:oae-frontend
      <th>u:oae:bert
      <th>u:oae:nicolaas
      <th>u:gat:stuartf
    <tr>
      <td>
      <td>member
      <td>member
      <td>member
</table>
 
# Group Membership

Group Membership for a given principal is managed using the Roles CF directly, by using a group as a resource, and assigning a principal a role on the group resource. This is done internally, so as to avoid the group-resource restrictions of the roles API. See group-related use-cases in the Roles CF for more information on how the requirements of group membership are fulfilled.

**Note about fault tolerance:** Adding a user membership to GroupMembers and Roles CF's is a batch operation, however there is still possibility for incomplete updates. That said, since the Roles CF determines the effective group membership for permissions checks, the GroupMembership CF should always have a membership added first, then Roles second. When removing membership, Roles CF entry should be deleted first, then GroupMembership second. In this case, you never end up with a case in which a user unknowingly has access to content while data is in an inconsistent state.
 
# Permissions

Individual permission actions are currently synonymous to roles. For example, if a user has the 'manager' role on a resource, it is the 'manager' permission that you would have to check for that user to see if they can manage. Currently, there is no feature to assign a number of individual actions to roles on a resource, as this permission model is currently not used in the application. It is likely one day it may, in which case some permission logic will have to change.

The difference between checking a permission for a principal+resource and checking a role between a principal+resource, is that permission checks are inherited VIA group memberships. Therefore if user **mrvisser** is a member of a group that has **manager** access on resource **Foo.docx**: Does mrvisser have **manager role** on **Foo.docx**? No; Does mrvisser have **manager permission** on **Foo.docx**? Yes.

A heavy permission check is to use the existing CF's, and group membership ancestry technique discussed in the Roles query use cases. The authz module will over the group membership ancestry to determine if one has the permission. This is done through a series of Cassandra 'multi-gets'.
