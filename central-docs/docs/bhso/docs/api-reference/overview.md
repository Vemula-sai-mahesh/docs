# **API Reference Overview**

Welcome to the BHSO API Reference. This section provides detailed information about all available API endpoints, their parameters, request/response formats, and example usage.

Before diving into specific resources, please review the following general information applicable to most API interactions.

## 1. Base URL

All API endpoints documented in this reference are relative to the following base URL:

`https://bookhealth-api.atparui.com/bookhealthServiceOnline`

For example, an endpoint documented as `/auth/login` would be accessed at `https://bookhealth-api.atparui.com/bookhealthServiceOnline/auth/login`.

## 2. API Versioning

[YOU WILL PROVIDE THIS - Check if your paths like `/api/v1/admin/users` in the "User Management System" doc indicate a versioning strategy. If so, describe it. If not explicitly versioned in the path, state that.]

Example if versioned in path:
"The BHSO API uses URL path versioning. The current stable version is `v1`, and you will see it prefixed in many resource paths (e.g., `/api/v1/admin/users`). Future API versions, if any, will be introduced with a new version prefix."

Example if not explicitly versioned yet:
"Currently, the API does not use explicit versioning in the URL paths. Any significant breaking changes in the future would likely be introduced under a new API endpoint structure or communicated through our [Changelog](../changelog.md)."

## 3. Authentication

Most API endpoints require authentication using a **JSON Web Token (JWT)**. Please refer to the detailed **[Authentication Guide](../authentication.md)** for instructions on how to obtain and use JWTs.

Authenticated requests must include an `Authorization` header:

Authorization: Bearer <YOUR_ACCESS_TOKEN>


Endpoints that do not require authentication (e.g., `/auth/login`, `/auth/register`, some public data endpoints if any) will be explicitly noted in their respective documentation.

## 4. Request Format

*   **Content-Type:** For requests that include a body (e.g., `POST`, `PUT`, `PATCH`), the API primarily expects `application/json`.
    ```
    Content-Type: application/json
    ```
*   **File Uploads:** For endpoints involving file uploads (e.g., doctor profile pictures, department banners), the API supports `multipart/form-data`. This will be specified on the relevant endpoint documentation. *(This is mentioned in section 3.3 of your "User Management System" doc).*

## 5. Response Format

The BHSO API uses a standardized JSON response structure for all requests, whether successful or an error, based on the `ApiResponse<T>` model.

### 5.1. Successful Responses
Successful responses will typically have an HTTP status code in the `2xx` range (e.g., `200 OK`, `201 Created`, `204 No Content`). The JSON body will follow this structure:

```
{
  "success": true,
  "message": "Descriptive success message.", // e.g., "User retrieved successfully."
  "data": { /* The actual resource data or null if no data */ },
  "timestamp": "YYYY-MM-DD HH:MM:SS" // Server timestamp of the response
}
```
* success: Always true for successful operations.
* message: A human-readable message indicating the outcome.
* data: The requested resource or result of the operation. This field may be null or omitted if the operation does not return data (e.g., for a successful DELETE or some PUT operations).
* timestamp: The server time when the response was generated.

### 5.2. Error Responses

Error responses will have an HTTP status code in the 4xx (client errors) or 5xx (server errors) range. The JSON body will also follow the ApiResponse<T> structure, but with success: false and additional error details:
```
{
"success": false,
"message": "Descriptive error message.", // e.g., "Validation failed", "Resource not found"
"errorCode": 12345, // An application-specific error code
"timestamp": "YYYY-MM-DD HH:MM:SS",
"validationErrors": [ // Present for validation failures (e.g., 400 Bad Request)
{
"field": "fieldName",
"message": "Specific validation error for this field."
}
// ... more validation errors if applicable
]
}
```
* success: Always false for error conditions.
* message: A human-readable summary of the error.
* errorCode: An application-specific integer code identifying the type of error. This can be useful for programmatic error handling by clients.
* timestamp: The server time when the response was generated.
* validationErrors (Optional): An array of objects detailing field-specific validation errors. Each object contains the field name and the message for that field. This is typically present for 400 Bad Request errors due to input validation failures.

For more detailed information on specific error codes and handling, please refer to the Error Handling Guide.

## 6. HTTP Methods
   The API utilizes standard HTTP methods to perform actions on resources:
   * GET: Retrieve a resource or a list of resources. Generally safe and idempotent.
   * POST: Create a new resource. Not idempotent.
   * PUT: Update an existing resource completely. Should be idempotent.
   * PATCH: Partially update an existing resource. May or may not be idempotent depending on the operation. [YOU WILL PROVIDE THIS: Confirm if PATCH is used by your API. If not, remove this bullet.]
   * DELETE: Remove a resource.
   
## 7. Common HTTP Status Codes
   While detailed error scenarios are covered in the Error Handling Guide, here are some common HTTP status codes you will encounter:

   * 200 OK: Standard response for successful GET, PUT, PATCH requests.
   * 201 Created: Response for successful POST requests that result in resource creation. The response body usually contains the created resource.
   * 204 No Content: Response for successful requests that do not return a body (e.g., successful DELETE operations).
   * 400 Bad Request: The request was malformed, contained invalid parameters, or failed business logic validation. The response body will often include validationErrors.
   * 401 Unauthorized: Authentication is required and has failed or has not been provided.
   * 403 Forbidden: The authenticated user does not have permission to perform the requested action on the resource.
   * 404 Not Found: The requested resource could not be found on the server.
   * 500 Internal Server Error: An unexpected error occurred on the server. These should be rare.

## 8. Pagination

   **YOU WILL PROVIDE THIS**: How is pagination handled for endpoints that return lists of resources (e.g., list of doctors, list of appointments)?
   Common methods include:

* Offset and Limit: e.g., ?offset=0&limit=20
* Page Number and Size: e.g., ?page=1&size=20
* Cursor-based pagination.

  Describe the query parameters used and how pagination information (e.g., total items, total pages) is returned in the response (often in the data object or as headers).
* 
  Example if using page and size:

  "For endpoints that return a list of items, pagination is supported using page and size query parameters.
  page: The page number to retrieve (1-indexed). Default: 1.
  size: The number of items per page. Default: 10, Max: 100.
  The response for paginated data will typically be structured within the data field of the ApiResponse as follows:
```
  {
  "success": true,
  "message": "Items retrieved successfully.",
  "data": {
  "content": [ /* array of items */ ],
  "pageable": {
  "pageNumber": 0, // 0-indexed page number in Spring Data JPA
  "pageSize": 10,
  "sort": { /* sort details */ },
  "offset": 0,
  "paged": true,
  "unpaged": false
  },
  "totalPages": 5,
  "totalElements": 48,
  "last": false,
  "size": 10,
  "number": 0, // current page number (0-indexed)
  "sort": { /* sort details */ },
  "numberOfElements": 10,
  "first": true,
  "empty": false
  },
  "timestamp": "..."
  }
```
 
  (If your API uses Spring Data JPA's Pageable, the response structure might look like the example above. Adjust based on your actual implementation.)
## 9. Sorting
   **YOU WILL PROVIDE THIS**: How is sorting handled for list endpoints?
   Common methods include:
* A sort query parameter, e.g., ?sort=fieldName,asc or ?sort=fieldName,desc.
* Multiple sort parameters.
  Describe the query parameters used.

  Example:
  "List endpoints may support sorting via the sort query parameter. The format is sort=property,direction.

  property: The field name to sort by (e.g., createdAt, name).

  direction: asc for ascending, desc for descending.

  Example: ?sort=lastName,asc&sort=firstName,asc"
## 10. Rate Limiting

**YOU WILL PROVIDE THIS OR REMOVE**: Is rate limiting implemented? If so, describe:

* The limits (e.g., requests per minute/hour per IP/user).

* How clients can identify their current rate limit status (e.g., X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset headers).

* The HTTP status code returned when limits are exceeded (typically 429 Too Many Requests).

  If not implemented, you can state: "Currently, the BHSO API does not enforce strict rate limiting. However, we monitor usage and reserve the right to implement rate limiting in the future to ensure fair usage and platform stability. Please use the API responsibly." 

## 11. Idempotency

The subsequent pages in this API Reference section are organized by resource or functional area (e.g., Authentication, Users, Departments, Doctors, Appointments). Each page will detail the specific endpoints available, including:

* HTTP Method and Path
* Description of the endpoint's purpose
* Required roles/permissions (if applicable beyond general authentication)
* Path and Query Parameters
* Request Body schema and examples
* Response schemas and examples for success and common error cases
* Example cURL requests

Please proceed to the specific resource pages for detailed endpoint information.

<!--
---------------------------------------------------------------------------------
TO-DO & VERIFICATION LIST FOR THIS PAGE (api-reference/overview.md):
---------------------------------------------------------------------------------
1. Section 2 (API Versioning):
* Confirm if the API uses URL path versioning (e.g., `/v1/`).
* If yes, ensure the statement is accurate and link to `versioning.md` is appropriate.
* If no, rephrase to state it's not explicitly versioned or how changes are handled.

2. Section 6 (HTTP Methods):
* Confirm if the `PATCH` HTTP method is used anywhere in the BHSO API. If not, remove the bullet point for `PATCH`.

3. Section 8 (Pagination):
* Is pagination implemented for list endpoints?
* If yes:
* What are the query parameters (e.g., `page`, `size`, `offset`, `limit`)?
* What is the structure of the response for paginated data (e.g., does it include `totalElements`, `totalPages`, `content` array, etc.)? Provide an example.
* If no, either state "Not currently implemented" or remove the section.

4. Section 9 (Sorting):
* Is sorting implemented for list endpoints?
* If yes:
* What is the query parameter (e.g., `sort=fieldName,direction`)?
* What are some example sortable fields for key resources?
* If no, either state "Not currently implemented" or remove the section.

5. Section 10 (Rate Limiting):
* Is rate limiting implemented?
* If yes:
* What are the limits?
* What HTTP headers are used to communicate rate limit status?
* What is the status code for exceeding limits (e.g., 429)?
* If no, the provided placeholder text is suitable.

6. Section 11 (Idempotency):
* Does the API offer any specific mechanisms for idempotency on `POST` requests (e.g., an `Idempotency-Key` header)? If so, this section should be expanded or specific endpoint docs should cover it. If not, the current general statement is okay.

Please gather this information from your development team or API specifications.
---------------------------------------------------------------------------------
-->