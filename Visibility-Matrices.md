## General Search

*Note: The concept of "joinable" doesn't really apply to users, content or discussions as of now, but the notion of "requesting access" to something other than a group could theoretically still be applied (e.g., if a user could request to connect with another user, or user could request access to content or discussion)*

<table>
  <thead>
    <tr>
      <td>
      <th colspan=3>Visibility
    <tr>
      <th>User Access
      <th>Public
      <th>Logged In
      <th>Private
  <tbody>
    <tr>
      <th>External User (another tenant, not member)
      <td>Returned
      <td>Returned if joinable = "yes" or "request", AND if both tenants are public. Not returned otherwise
      <td>Returned if joinable = "yes" or "request", AND if both tenants are public. Not returned otherwise
    <tr>
      <th>Anonymous
      <td>Returned
      <td>Not Returned
      <td>Not Returned
    <tr>
      <th>Logged In (no membership and not self)
      <td>Returned
      <td>Returned
      <td>Returned if joinable = "yes" or "request". Not returned otherwise
    <tr>
      <th>Member (or "self" in case of user)
      <td>Returned
      <td>Returned
      <td>Returned
</table>
