# Overview

This page contains recommendations for implementing authorization.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- You can use Role-Based Access Control (RBAC) approach to implement authorization if having a particular role by a user is sufficient to grant access.

<details>
<summary>Clarification</summary>

RBAC can be used to implement linear access control where the accessibility of an object or method can be determined by the presence of a particular role.

Imagine an application for hosting information about a certain product, where there are three types of users:

- `Anonymous user` can view information about a product.
- `Editor` can edit information about a product.
- `Admin` has full access to an application and can configure both content and an application itself.

In this case, authorization is quite linear, all that is needed when performing a certain action is to make sure that a user has a correct role.

At the code level, this looks like a validation for the presence of an expected role in each controller before processing a request from a client.

```go
func editContentController(content) {
    if current_user.role < Roles.EDITOR {
        return "Access denied", 401
    }

    update(content)

    return "OK", 200
}
```
</details>

- Use Attribute-Based Access Control (ABAC) approach to implement authorization if it is necessary to provide more granular access.

<details>
<summary>Clarification</summary>

ABAC can be used when you need to grant access based on more criteria than just having a role.

Imagine we have a data model containing vehicles and users, and we want to know if a user can drive a vehicle. In this case the attributes can be the following ones:

- `owns` - Is a user an owner of a vehicle?
- `has_access_to` - Does a user have permission from an owner to drive a vehicle?
- `old_enough_to_drive` - Is it legal to drive a vehicle at a user's age?
- `has_driving_license` - Does a user have a driving license?

These attributes allow you to define a policy for deciding whether a user can drive a vehicle or not:

```go
drive_vehicle = (owns | has_access_to) & old_enough_to_drive & has_driving_license
```

At the code level, this looks like checking if a particular action can be performed before it is performed:

```go
func driveController(vehicle) {
    if !can(currentUser, "drive_vehicle", vehicle) {
        return "Access denied", 401
    }

    drive(vehicle)

    return "OK", 200
}
```
</details>

- Give the minimum level of permissions that is necessary to complete an assigned operation.

<details>
<summary>Example</summary>

- Even though both an accountant and a sales representative occupy the same level in an organization's hierarchy, both require access to different resources to perform their jobs. The accountant should likely not be granted access to a customer database and the sales representative should not be able to access payroll data.
- A sales representative should only have access to the data of the customers with whom they work.
</details>

- Give permissions only for a minimum amount of time that is necessary to complete the operation.

<details>
<summary>Example</summary>

- If there is a group of users who have access to certain resources, removing a user from that group should revoke permissions to those resources.
- API token must have an expiration date to reduce the time to perform a potential attack in case a token is leaked to source code or shared resources.
</details>

- Deny access by default. Grant required permissions only after successfully validating an access control policy.

<details>
<summary>Example</summary>

**Note**: The following example is vulnerable, do **not** use it as a reference example!

Consider the following method that checks admin permissions for a current user:

```js
func isAdmin() {
    isAdmin = true
    try {
        // some actions that can lead to error
        if current_user.role < Roles.ADMIN {
        isAdmin = false
    }
    } catch (error) {
        log.warn("Error message: " + error)
    }

    return isAdmin
}
```

In the snippet above, the `isAdmin` method sets the `isAdmin` variable to `true` by default. Since we consider a current user as an admin by default, an error handling pitfall can lead to an access control bypass. Therefore, if a user can trigger an error before checking their role, they can elevate their privileges.

Deny access by default to mitigate/avoid the effects of such pitfalls.
</details>

- Check permissions on every action. Make sure that a user or process has the necessary permissions before performing any action.
- Take a consistent permission validation approach and implement it at the same application level. Do **not** spread this functionality over different levels.

<details>
<summary>Clarification</summary>

Develop an access control policy as a separate component that can be used by other components to validate permissions. Check permissions with this component, for example, inside controllers. Try to avoid checking permissions in multiple layers, for example, controllers and the data layer.
</details>

- Create unit and integration tests that validate enrolled access control policy. Make sure that the permissions mapped out in accordance with business flows are being correctly enforced.
- Avoid exposing identifiers to users when possible. For example, use `/user/orders` instead of `/users/1337/orders` to fetch orders for a current user.
- Do **not** provide access based on knowledge of an entity ID. Even if an ID is long and random enough to be unguessable, always check permissions before providing access.
- Make sure static content is covered by access control policy when appropriate.
- Handle all exceptions, errors and authorization failures no matter how unlikely they seem, see the [Error and Exception Handling](/Web%20Application/Error%20and%20Exception%20Handling/README.md) page.
- Log authorization attempts, at least authorization failures and operations requiring elevated privileges, see the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Regularly review an access control policy to avoid "privilege creep" and possible abuse of a policy for privilege escalation.
- Document an access control policy. It must include each user role, actions that an application supports, and permissions that are required to perform those actions.
