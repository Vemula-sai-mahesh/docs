# Users API

The Users API provides endpoints for managing user accounts and profiles within the BHSO platform.

## Common DTOs

### UserDto 
Represents a user in the system.

```
{
  "id": "c9e8f8e0-9b1a-4b1c-8f4c-8c1b1e4a5e2a",
  "firstName": "John",
  "lastName": "Doe",
  "middleName": "M",
  "gender": "MALE",
  "dateOfBirth": "1990-01-15",
  "email": "john.doe@example.com",
  "role": "ROLE_PATIENT", // e.g., ROLE_PATIENT, ROLE_DOCTOR, ROLE_TENANT_ADMIN, ROLE_SUPER_ADMIN
  "contactNo": "1234567890",
  "bloodGroup": "O_POSITIVE",
  "address": "123 Main St, Anytown",
  "active": true,
  "createdAt": "2023-10-01T10:00:00Z",
  "updatedAt": "2023-10-01T10:00:00Z"
  // Any other fields present in your UserDto
}
```
### PasswordResetRequestDto

Used for resetting a user's password.
```
{
"newPassword": "NewSecurePassword123!"
}
```
### SendOtpRequest

Used to request an OTP for an email address.
```
{
"email": "user@example.com"
}
```
### VerifyOtpRequest
Used to verify an OTP sent to an email address.
```
{
"email": "user@example.com",
"otp": "123456"
}
```
## **Get Current User Details**

Retrieves the profile information of the currently authenticated user.

* Endpoint: **GET /api/users/me**
    * Authentication: Required (Bearer Token)
    * Description: Fetches details of the logged-in user.
    * Responses:

        * 200 OK: Successfully retrieved user details.
        * Body: UserDto (See Common DTOs above)
        * 401 Unauthorized: Authentication token is missing or invalid.
        * 404 Not Found: User associated with the token not found (should be rare if token is valid).

## **Update Current User Profile**
Allows the currently authenticated user to update their profile information.

* Endpoint: **PUT /api/users/profile**
    * Authentication: Required (Bearer Token)
    * Permissions: Authenticated User (Any role)
    * Description: Updates mutable fields of the logged-in user's profile.
    * Request Body: UpdateUserProfileRequestDto
    ```
    {
    "firstName": "Johnathan",
    "lastName": "Doer",
    "middleName": "Michael",
    "gender": "MALE",
    "dateOfBirth": "1990-01-15",
    "contactNo": "0987654321",
    "bloodGroup": "A_NEGATIVE",
    "address": "456 New Ave, Othertown"
    }
  
    ```
    * Responses:
        * 200 OK: Profile updated successfully.
        * Body: UserDto (The updated user profile)
        * 400 Bad Request: Invalid input data.
        * *401 Unauthorized: Authentication token is missing or invalid.
* Example cURL:
```
curl -X PUT "https://bookhealth-api.atparui.com/bookhealthServiceOnline/api/users/profile" \
-H "Authorization: Bearer <YOUR_ACCESS_TOKEN>" \
-H "Content-Type: application/json" \
-d '{
"firstName": "Johnathan",
"contactNo": "0987654321"
}'
```
## Get User by ID
Retrieves the profile information of a specific user by their ID.

* Endpoint: GET /api/users/{userId}
* Authentication: Required (Bearer Token)
* Permissions: [ASSUMPTION TO BE VERIFIED: TENANT_ADMIN, SUPER_ADMIN]
* Path Parameters:
    * userId (string, UUID): The unique identifier of the user to retrieve.
* Description: Fetches details of a specific user.
* Responses:

    * 200 OK: Successfully retrieved user details.
    * Body: UserDto
    * 401 Unauthorized: Token missing or invalid.
    * 403 Forbidden: Insufficient permissions.
    * 404 Not Found: User with the specified userId not found.
  
Example cURL:
```
curl -X GET "https://bookhealth-api.atparui.com/bookhealthServiceOnline/api/users/c9e8f8e0-9b1a-4b1c-8f4c-8c1b1e4a5e2a" \
-H "Authorization: Bearer <YOUR_ADMIN_ACCESS_TOKEN>"
```

## Update User Status
Updates the active/inactive status of a specific user.

* Endpoint: PUT /api/users/{userId}/status
* Authentication: Required (Bearer Token)
* Permissions: [ASSUMPTION TO BE VERIFIED: TENANT_ADMIN, SUPER_ADMIN]
* Path Parameters:
    * userId (string, UUID): The unique identifier of the user whose status is to be updated.
* Request Body: UserStatusUpdateRequestDto
```
{
"active": false
}
```
* Description: Activates or deactivates a user account.
* Responses:

    * 200 OK: User status updated successfully.
    * Body: UserDto (The updated user profile)
    * 400 Bad Request: Invalid input.
    * 401 Unauthorized: Token missing or invalid.
    * 403 Forbidden: Insufficient permissions.
    * 404 Not Found: User with userId not found.

* Example cURL:
```
curl -X PUT "https://bookhealth-api.atparui.com/bookhealthServiceOnline/api/users/c9e8f8e0-9b1a-4b1c-8f4c-8c1b1e4a5e2a/status" \
-H "Authorization: Bearer <YOUR_ADMIN_ACCESS_TOKEN>" \
-H "Content-Type: application/json" \
-d '{
"active": false
}'
```

## Delete User by ID
Deletes a specific user by their ID.

* Endpoint: DELETE /api/users/{userId}
* Authentication: Required (Bearer Token)
* Permissions: [ASSUMPTION TO BE VERIFIED: TENANT_ADMIN, SUPER_ADMIN]
* Path Parameters:

    * userId (string, UUID): The unique identifier of the user to delete.
  
* Description: Permanently deletes a user account. Use with caution.
  * Responses:

      * 200 OK (or 204 No Content): User deleted successfully. Response body may be a success message or empty.
  
          ```
            // Example if a message is returned
            {
            "success": true,
            "message": "User deleted successfully."
            }
          ```
  
      * 401 Unauthorized: Token missing or invalid.
  
      * 403 Forbidden: Insufficient permissions.
  
      * 404 Not Found: User with userId not found.
* Example cURL:
```
  curl -X DELETE "https://bookhealth-api.atparui.com/bookhealthServiceOnline/api/users/c9e8f8e0-9b1a-4b1c-8f4c-8c1b1e4a5e2a" \
  -H "Authorization: Bearer <YOUR_ADMIN_ACCESS_TOKEN>"
```

## Reset User Password
Allows an administrator to reset the password for a specific user.

- Endpoint: PUT /api/users/{userId}/reset-password
- Authentication: Required (Bearer Token)
- Permissions: [ASSUMPTION TO BE VERIFIED: TENANT_ADMIN, SUPER_ADMIN]
- Path Parameters:

  - userId (string, UUID): The unique identifier of the user whose password is to be reset.
- Request Body: PasswordResetRequestDto
```
{
"newPassword": "NewStrongPassword123!"
}
```
- Description: Sets a new password for the specified user. The user should be informed to change this password upon their next login if it's admin-initiated.
  - Responses:

      - 200 OK: Password reset successfully. Response body might be a success message or the updated UserDto.
```
// Example if a message is returned
{
"success": true,
"message": "Password reset successfully for user."
}
```
   - 400 Bad Request: Invalid newPassword (e.g., fails complexity rules) or other validation issues.
   - 401 Unauthorized: Token missing or invalid. 
   - 403 Forbidden: Insufficient permissions. 
   - 404 Not Found: User with userId not found.
- Example cURL:
```
curl -X PUT "https://bookhealth-api.atparui.com/bookhealthServiceOnline/api/users/c9e8f8e0-9b1a-4b1c-8f4c-8c1b1e4a5e2a/reset-password" \
-H "Authorization: Bearer <YOUR_ADMIN_ACCESS_TOKEN>" \
-H "Content-Type: application/json" \
-d '{
"newPassword": "NewStrongPassword123!"
}'
```
## Get All Users
Retrieves a list of all users. This endpoint may support pagination.

- Endpoint: GET /api/users
- Authentication: Required (Bearer Token)
- Permissions: [ASSUMPTION TO BE VERIFIED: TENANT_ADMIN, SUPER_ADMIN]
- Query Parameters (Pagination - [ASSUMPTION TO BE VERIFIED]):
    - page (integer, optional): Page number.
    - size (integer, optional): Users per page.
    - sort (string, optional): Sort order.
- Description: Fetches a list of all users.
- Responses:
      - 200 OK: Successfully retrieved list of users.
          - Body: Array of UserDto or paginated response containing UserDto array.
      - 401 Unauthorized: Token missing or invalid.
      - 403 Forbidden: Insufficient permissions.
- Example cURL:
```
curl -X GET "https://bookhealth-api.atparui.com/bookhealthServiceOnline/api/users?page=0&size=10" \
-H "Authorization: Bearer <YOUR_ADMIN_ACCESS_TOKEN>"
```

## Get Users by Role
Retrieves a list of users filtered by a specific role.

- Endpoint: GET /api/users/by-role/{roleName}
- Authentication: Required (Bearer Token)
- Permissions: [ASSUMPTION TO BE VERIFIED: TENANT_ADMIN, SUPER_ADMIN]
- Path Parameters:

     roleName (string): The role to filter by (e.g., PATIENT, DOCTOR).

- Query Parameters (Pagination - [ASSUMPTION TO BE VERIFIED]):

page (integer, optional): Page number.

size (integer, optional): Users per page.

- Description: Fetches users belonging to the specified role.
- Responses:

    - 200 OK: Successfully retrieved list of users.
    - Body: Array of UserDto or paginated response. 
    - 400 Bad Request: Invalid roleName. 
    - 401 Unauthorized: Token missing or invalid. 
    - 403 Forbidden: Insufficient permissions.
  
- Example cURL:
```
curl -X GET "https://bookhealth-api.atparui.com/bookhealthServiceOnline/api/users/by-role/DOCTOR?size=5" \
-H "Authorization: Bearer <YOUR_ADMIN_ACCESS_TOKEN>"
```

## Get Users by Tenant ID
Retrieves a list of users associated with a specific tenant.

- Endpoint: GET /api/users/tenant/{tenantId}
- Authentication: Required (Bearer Token)
- Permissions: [ASSUMPTION TO BE VERIFIED: SUPER_ADMIN, or TENANT_ADMIN for their own tenantId]
- Path Parameters:

     - tenantId (string, UUID): The unique identifier of the tenant.
  
- Query Parameters (Pagination - [ASSUMPTION TO BE VERIFIED]):
     - page (integer, optional): Page number.
     - size (integer, optional): Users per page.
- Description: Fetches users belonging to the specified tenant.
- Responses:
     - 200 OK: Successfully retrieved list of users.
     - Body: Array of UserDto or paginated response.
     - 401 Unauthorized: Token missing or invalid.
     - 403 Forbidden: Insufficient permissions. 
     - 404 Not Found: Tenant with tenantId not found. 
- Example cURL:
```
curl -X GET "https://bookhealth-api.atparui.com/bookhealthServiceOnline/api/users/tenant/tnt-uuid-001?size=20" \
-H "Authorization: Bearer <YOUR_SUPER_ADMIN_ACCESS_TOKEN>"
```

## Send OTP for Password Reset
Initiates a password reset process by sending an OTP to the user's registered email.
- Endpoint: POST /api/users/send-otp 
- Authentication: Not Required (Public endpoint for forgotten password scenarios)
- Description: Sends a One-Time Password (OTP) to the provided email address if it exists in the system, for the purpose of password recovery. 
- Request Body: SendOtpRequest
```
{
"email": "user.to.reset@example.com"
}
```

- Responses:
     - 200 OK: OTP sent successfully (or a generic message indicating process initiated, to avoid leaking user existence).
```
// Example response
{
"success": true,
"message": "If your email address is registered, an OTP has been sent."
}
```
- 400 Bad Request: Invalid email format.
- 404 Not Found: (Potentially) If email is not found, though often a generic success is returned for security. [BEHAVIOR TO BE VERIFIED]
- Example cURL:
```bash
curl -X POST "https://bookhealth-api.atparui.com/bookhealthServiceOnline/api/users/send-otp" \
-H "Content-Type: application/json" \
-d '{
"email": "user.to.reset@example.com"
}'
```

## Verify OTP for Password Reset
Verifies the OTP provided by the user for password reset. This step usually precedes allowing the user to set a new password.

- Endpoint: POST /api/users/verify-otp
- Authentication: Not Required
- Description: Verifies an OTP that was sent to a user's email for password reset.
- Request Body: VerifyOtpRequest
```json
{
"email": "user.to.reset@example.com",
"otp": "123456"
}
```
- Responses:

     200 OK: OTP verified successfully. The response might include a temporary token or flag to allow the subsequent password change.
```
// Example response - actual structure may vary
{
"success": true,
"message": "OTP verified successfully. You can now reset your password."
// "resetToken": "some_temporary_token_for_password_reset" // If applicable
}
```
400 Bad Request: OTP is invalid, expired, or email does not match an active OTP request.
- Example cURL:
```bash
curl -X POST "https://bookhealth-api.atparui.com/bookhealthServiceOnline/api/users/verify-otp" \
-H "Content-Type: application/json" \
-d '{
"email": "user.to.reset@example.com",
"otp": "123456"
}'
```
Note: After successful OTP verification, there is typically another endpoint for the user to set their new password, often requiring the email and the new password (and potentially the temporary resetToken from the verify step). This "set new password after OTP" endpoint is not listed under user-controller in your Swagger; it might be under auth-controller or a dedicated password reset controller. [ENDPOINT TO BE IDENTIFIED & DOCUMENTED IF PART OF THIS FLOW]

<!--
---------------------------------------------------------------------------------
TO-DO & VERIFICATION LIST FOR THIS PAGE (api-reference/users.md):
---------------------------------------------------------------------------------
1. Permissions: For each endpoint, verify and specify the exact user roles (e.g., `TENANT_ADMIN`, `SUPER_ADMIN`, Authenticated User) that are authorized to use it.
2. Pagination & Sorting for List Endpoints:
* For `GET /api/users`, `GET /api/users/by-role/{roleName}`, `GET /api/users/tenant/{tenantId}`:
* Confirm if pagination and sorting are implemented.
* If yes, provide the exact query parameters and the structure of the paginated response.
3. `POST /api/users/send-otp` (404 Behavior): Confirm if a generic success message is returned even if the email is not found (for security), or if a 404 is returned.
4. "Set New Password after OTP" Endpoint: Identify and document the endpoint used to set a new password after OTP verification if it's part of this user-initiated password reset flow.
5. `POST /api/users/logout`:
* Confirm which token (`accessToken` or `refreshToken`) is expected in the `LogoutRequestDto`.
* Confirm server-side invalidation behavior.
* Confirm success response structure (Swagger implies no body).
6. `/api/v1/admin/users/*` Endpoints: Clarify if these are distinct, current, and need to be documented here or separately.
---------------------------------------------------------------------------------
-->