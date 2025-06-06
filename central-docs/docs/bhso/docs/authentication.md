# Authentication

Access to the BHSO API is protected and requires authentication for most endpoints. The platform utilizes **JSON Web Tokens (JWT)** to ensure secure communication. All API requests should be made over **HTTPS**.

This guide explains how to obtain, use, and manage JWTs for interacting with the BHSO API.

## 1. Authentication Overview

The BHSO API employs a token-based authentication mechanism. After a user successfully authenticates via the `/auth/login` endpoint, the API issues an `accessToken` and a `refreshToken`. The `accessToken` must be included in the `Authorization` header of subsequent requests to protected resources, using the `Bearer` scheme.

Example: `Authorization: Bearer <YOUR_ACCESS_TOKEN>`

## 2. User Roles & Authentication Scopes

The BHSO platform supports multiple user roles, as defined during registration (e.g., `PATIENT`, `DOCTOR`, `TENANT_ADMIN`, `SUPER_ADMIN`). The JWT issued to a user will contain claims reflecting their role and associated permissions. The API enforces authorization based on these claims, ensuring users can only access resources and perform actions appropriate for their role.

*   Attempting to access an endpoint without a valid token will result in a `401 Unauthorized` error.
*   Attempting to perform an action for which the authenticated user does not have permission will result in a `403 Forbidden` error.

## 3. Obtaining an Access Token

### 3.1. User Registration
New users can create an account using the registration endpoint.

*   **Endpoint:** `POST /auth/register`
*   **Request Body:** `RegisterRequest`
```json
{
  "firstName": "John",
  "lastName": "Doe",
  "middleName": "M",
  "gender": "MALE",
  "dateOfBirth": "1990-01-15",
  "email": "john.doe@example.com",
  "password": "SecurePassword123!",
  "role": "PATIENT",
  "contactNo": "1234567890",
  "bloodGroup": "O_POSITIVE",
  "address": "123 Main St, Anytown"
}
```
*   **Success Response (200 OK):** A string message.
```text
"User registered successfully. Please login to continue."
```
*   **Error Response:** Standard API error structure if validation fails or user already exists.
*   **Example cURL:**
```bash
    curl -X POST "https://bookhealth-api.atparui.com/bookhealthServiceOnline/auth/register" \
         -H "Content-Type: application/json" \
         -d '{
               "firstName": "John", "lastName": "Doe", "email": "john.doe@example.com",
               "password": "SecurePassword123!", "role": "PATIENT",
               "dateOfBirth": "1990-01-15", "gender": "MALE"
             }'
```
After successful registration, the user can proceed to log in.

### 3.2. User Login
Registered users can obtain an `accessToken` and `refreshToken` by logging in.

*   **Endpoint:** `POST /auth/login`
*   **Request Body:** `LoginRequest`
```json
    {
      "email": "john.doe@example.com",
      "password": "SecurePassword123!"
    }
```
*   **Success Response (200 OK):** `AuthResponse`
```json
    {
      "accessToken": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJqb2huLmRvZUBleGFtcGxlLmNvbSIsInJvbGUiOiJST0xFX1BBVElFTlQiLCJ1c2VySWQiOiJhYmMxMjMifeJ0aSI6ImQzNjU5ODBjLTU1MzYtNGIyZS04YjYyLTdkZWI2ZjdkZGRiOSIsImlhdCI6MTcwMDAwMDAwMCwiZXhwIjoxNzAwMDAzNjAwfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
      "refreshToken": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhYmMxMjM0Iiwicm9sZSI6IlJPTEVfUEFUSUVOVCIsInVzZXJJZCI6ImFiYzEyMzQiLCJqdGkiOiJkMzY1OTgwYy01NTM2LTRiMmUtOGI2Mi03ZGViNmY3ZGRkYjkiLCJpYXQiOjE3MDAwMDAwMDAsImV4cCI6MTcwMDYwNDgwMH0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
      "userId": "abc12345-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "role": "ROLE_PATIENT",
      "email": "john.doe@example.com"
    }
```
*   **Error Response (e.g., 401 Unauthorized):** Standard API error structure for invalid credentials.
*   **Example cURL:**
```bash
    curl -X POST "https://bookhealth-api.atparui.com/bookhealthServiceOnline/auth/login" \
         -H "Content-Type: application/json" \
         -d '{
               "email": "john.doe@example.com",
               "password": "SecurePassword123!"
             }'
```

## 4. Token Types & Lifespan

The BHSO API issues two types of tokens:

*   **Access Token (`accessToken`):**
    *   **Purpose:** This token is sent in the `Authorization` header for every API request to access protected resources.
    *   **Lifespan:** **[ASSUMPTION TO BE VERIFIED]** Access tokens are short-lived, typically expiring in **1 hour**.
*   **Refresh Token (`refreshToken`):**
    *   **Purpose:** A longer-lived token used to obtain a new `accessToken` once the current one expires, without requiring the user to re-enter their credentials.
    *   **Lifespan:** **[ASSUMPTION TO BE VERIFIED]** Refresh tokens have a longer lifespan, typically **7 days**.
    *   **Storage:** Refresh tokens must be stored securely by the client application.

## 5. Using the Access Token

Once you have an `accessToken`, include it in the `Authorization` header of your API requests using the `Bearer` scheme:

Authorization: Bearer <YOUR_ACCESS_TOKEN>

If the token is missing, invalid, or expired, the API will respond with a 401 Unauthorized error.

## 6. Refreshing an Access Token

When your accessToken expires, use the refreshToken to obtain a new accessToken.

*  **Endpoint:** `POST /auth/refresh-token`
*  **Request Body: RefreshTokenRequest**

```
    {
"refreshToken": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhYmMxMjM0Iiwicm9sZSI6IlJPTEVfUEFUSUVOVCIsInVzZXJJZCI6ImFiYzEyMzQiLCJqdGkiOiJkMzY1OTgwYy01NTM2LTRiMmUtOGI2Mi03ZGViNmY3ZGRkYjkiLCJpYXQiOjE3MDAwMDAwMDAsImV4cCI6MTcwMDYwNDgwMH0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
}
```

*  **Success Response (200 OK): AuthResponse (containing a new accessToken).**

```
    {
"accessToken": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJqb2huLmRvZUBleGFtcGxlLmNvbSIsInJvbGUiOiJST0xFX1BBVElFTlQiLCJ1c2VySWQiOiJhYmMxMjM0IiwianRpIjoiZTQ3NjA5MWMtNjYzNy00YzNmLTk5YzAtOGNmZjdjOGVkZGMwIiwiaWF0IjoxNzAwMDAzNjAwLCJleHAiOjE3MDAwMDcyMDB9Q.TgmKyxRKTMeLLF3RU5gxrMeKg47QPk7zKW_beRttx6d",
"refreshToken": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhYmMxMjM0Iiwicm9sZSI6IlJPTEVfUEFUSUVOVCIsInVzZXJJZCI6ImFiYzEyMzQiLCJqdGkiOiJkMzY1OTgwYy01NTM2LTRiMmUtOGI2Mi03ZGViNmY3ZGRkYjkiLCJpYXQiOjE3MDAwMDAwMDAsImV4cCI6MTcwMDYwNDgwMH0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c", // [ASSUMPTION TO BE VERIFIED: The API may or may not implement refresh token rotation. If it does, a new refreshToken would be issued here. If not, the original refreshToken remains valid.]
"userId": "abc12345-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
"role": "ROLE_PATIENT",
"email": "john.doe@example.com"
}
```

Clients should attempt to refresh an accessToken upon receiving a 401 Unauthorized error. If the refresh attempt also fails, the user must re-authenticate.

## 7. Token Validation (Optional Client-Side Check)

The API provides an endpoint to validate a token. [ASSUMPTION TO BE VERIFIED: This is typically used by clients on startup to check if a stored token is still active before attempting to use it, or for server-to-server validation scenarios. For most client operations, attempting to use the token and handling a 401 error is the primary validation method.]

*  **Endpoint:** `GET /auth/validate`
*  **Query Parameter:** token (the accessToken to validate)
*  **Example Request:** GET https://bookhealth-api.atparui.com/bookhealthServiceOnline/auth/validate?token=your_access_token_here

* **Success Response (200 OK): `ValidateTokenResponse`**

```json
{
  "status": true,
  "message": "Token is valid",
  "user": {
    "id": "abc12345-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@example.com",
    "role": "ROLE_PATIENT"
  }
}
```

*  **Response if Invalid (200 OK with status:false):**

```
    {
    "status": false,
    "message": "Token is invalid or expired",
    "user": null
    }
```

## 8. Token Invalidation / Logout

To log out a user, the client should discard the stored accessToken and refreshToken. Additionally, the API provides an endpoint that may perform server-side invalidation of the token.

*  **Endpoint:** `POST /api/users/logout`
*  **Request Body:** LogoutRequestDto
```
    {
    "token": "your_current_access_token_here" // [ASSUMPTION TO BE VERIFIED: This is likely the accessToken being logged out.]
    }
```

*  **Authorization: [ASSUMPTION TO BE VERIFIED]** This endpoint likely requires the Authorization: Bearer <accessToken> header of the user who is logging out.
*  **Success Response (200 OK):** Typically a success message.

// Example - Confirm actual response structure. Your Swagger doc for this endpoint doesn't specify a response schema, implying it might be a simple 200 OK with no body or a generic success message.
```
    {
    "success": true,
    "message": "User logged out successfully."
    }
```

*  **Behavior: [ASSUMPTION TO BE VERIFIED]** This endpoint attempts to invalidate the provided token on the server-side (e.g., by adding it to a denylist until its natural expiry). It may or may not invalidate an associated refreshToken.
*  **Example cURL:**

```
    curl -X POST "https://bookhealth-api.atparui.com/bookhealthServiceOnline/api/users/logout" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer your_current_access_token" \
    -d '{
    "token": "your_current_access_token"
    }'
```

Regardless of the server response for the /api/users/logout endpoint, client applications must securely clear/discard both the accessToken and refreshToken from their storage upon user logout.