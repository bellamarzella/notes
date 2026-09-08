
| Code | Status             | Description                                                                        | Explanation                                                                        |
| ---- | ------------------ | ---------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| 301  | Moved Permenantly  | Requested resource has been permenantly moved to the URL in the `location` header. | The page redirects.                                                                |
| 401  | Unauthorised       | The client lacks the valid authentication for the requested resource.              | The server does not know who you are as your credentials are missing or invalid.   |
| 403  | Forbidden          | The server understood the request but refused to process it.                       | The server knows who you are and that you don't have permission to view this page. |
| 405  | Method Not Allowed | The server knows the request method, but the target does not support it.           | The page exists, but you are trying to interact with it incorrectly.               |
