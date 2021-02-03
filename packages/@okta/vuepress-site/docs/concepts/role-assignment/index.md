---
title: Role Assignment
---
<ApiLifecycle access="beta" />

# Role Assignment
Role assignment to principals makes them administrators of your org.

## Standard Role Assignment
The following role types are provided and supported out of the box:
| Role type                               | Label                               | Optional targets                      |
| :-------------------------------------- | :---------------------------------- | :------------------------------------ |
| `API_ACCESS_MANAGEMENT_ADMIN`           | API Access Management Administrator |                                       |
| `APP_ADMIN`                             | Application Administrator           | Apps                                  |
| `GROUP_MEMBERSHIP_ADMIN`                | Group Membership Administrator      | [Groups](/docs/reference/api/groups/) |
| `HELP_DESK_ADMIN`                       | Help Desk Administrator             | [Groups](/docs/reference/api/groups/) |
| `MOBILE_ADMIN`                          | Mobile Administrator                |                                       |
| `ORG_ADMIN`                             | Organizational Administrator        |                                       |
| `READ_ONLY_ADMIN`                       | Read-Only Administrator             |                                       |
| `REPORT_ADMIN`                          | Report Administrator                |                                       |
| `SUPER_ADMIN`                           | Super Administrator                 |                                       |
| `USER_ADMIN`                            | Group Administrator                 | [Groups](/docs/reference/api/groups/) |

Standard role assignment is done in two steps:
1- Assign a role to a user or group. At this point the admin has the supported privileges fo the role over all resources across organization.
2- (Optional) If the role supports targets, use one of the [target operations](/docs/reference/api/roles/#role-target-operations) to specify which specific resource the admin can manage.

Note that the entities involved in standard role assignment are:
* a role; identified either by type or id returned from [listing API](/docs/reference/api/roles/#list-roles)
* a principal; either a group or a user
* (Optional) a resource; when using target [target operations](/docs/reference/api/roles/#role-target-operations) this could be either an app or a group

## Custom Role Assignment
Custom roles can be built by piecing [permissions](/docs/reference/api/roles/#permission-types) together. Once a custom role is built, you can use its id or name to assign to admins. The process is:
1- Build a custom role
2- Build a set of resources
3- Bind the admin with the role from step 1 targeting the resource set from step 2

An assignment of a role to an admin is called a [binding](/docs/reference/api/roles/#binding-object). Conceptually, a binding is identified by the role id. A maximum of one role per resource set will be allowed. Assignment of more admins to the same role targetting the same resource set is possible and they are added as [members]((/docs/reference/api/roles/#member-object)) of the same binding. Hence a binding could simply be seen as:
```json
{
  "${roleId}": [
    "${memberId1}",
    "${memberId2}",
    ...
  ]
}
```

Therefore, when dealing with custom roles, three entities are always in play:
* a role; identified by its name or id
* a principal; either a group or a user
* a resource-set; identified by its id

### Resource Sets
Custom roles can only be assigned if scoped to a set of recourses from the beginning. In other words, you need to have a resource set in order to be able to assign a custom role.

A resource set is simply a collection of resources. The following resources are currently supported:
* All users
* All groups
* A specific group
* All users within a specific group
Note that the distinction between users and groups allows for creating custom admins that are only scoped to users or groups.

### Identifiers
#### Resource Identifiers
In order to specify a resource targeted by a resource set, you will simply use the REST URL of the corresponding Okta API:
* All users: `https://${yourOktaDomain}/api/v1/users`
* All groups: `https://${yourOktaDomain}/api/v1/groups`
* A specific group: `https://${yourOktaDomain}/api/v1/groups/${targetGroupId}`
* All users within a specific group: `https://${yourOktaDomain}/api/v1/groups/${targetGroupId}/users`

> **Tip:** If you use a role with permissions that don't apply to the resources in the resource set, the admin role will have no effect. For example, the `okta.users.profile.manage` permission will give the admin no privileges if granted to a resource set that only included `https://${yourOktaDomain}/api/v1/groups/${targetGroupId}` resources. If you want the admin to be able to manage the users within the group the resource set must include the corresponding `https://${yourOktaDomain}/api/v1/groups/${targetGroupId}/users` resource.

#### Member Identifiers
In order to specify binding members, you will simply use the REST URL of the corresponding Okta API:
* A specific user: `https://${yourOktaDomain}/api/v1/users/${memberUserId}`
* A specific group: `https://${yourOktaDomain}/api/v1/groups/${memberGroupId}`

## Custom vs. Standard
1- An admin can have both types of standard role assignments and custom role bindings. Note that privileges granted to an admin are an aggregate of:
* standard roles directly assigned
* standard roles granted through group membership
* custom roles directly assigned
* custom roles granted through group membership
As a result, if an admin was granted a standard role limited to a single group and at the same time received group management privileges on all groups in the org through a custom org, the ultimate outcome is group management on all groups.

2- Not all permissions and resources are currently supported by custom roles.

3- A custom role cannot be assigned without a resource set.
