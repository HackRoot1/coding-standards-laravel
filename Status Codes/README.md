# 📘 HTTP Status Codes (200–504)

A complete reference for common HTTP response codes, categorized with meanings and real-world use cases.

---

| Code | Category | Meaning | When You See It |
|------|-----------|----------|-----------------|
| **200** | ![Success](https://img.shields.io/badge/Success-2xx-brightgreen) | OK | The request was successful and the server returned the requested data. |
| **201** | ![Success](https://img.shields.io/badge/Success-2xx-brightgreen) | Created | A new resource has been successfully created (e.g., after a POST request). |
| **202** | ![Success](https://img.shields.io/badge/Success-2xx-brightgreen) | Accepted | The request was received but has not yet been processed. |
| **203** | ![Success](https://img.shields.io/badge/Success-2xx-brightgreen) | Non-Authoritative Information | The returned data is from a third-party source, not the original server. |
| **204** | ![Success](https://img.shields.io/badge/Success-2xx-brightgreen) | No Content | The server processed the request but has no content to return. |
| **205** | ![Success](https://img.shields.io/badge/Success-2xx-brightgreen) | Reset Content | The client should reset the document view (e.g., form submission). |
| **206** | ![Success](https://img.shields.io/badge/Success-2xx-brightgreen) | Partial Content | The server is returning part of a resource due to a range header. |
| **300** | ![Redirect](https://img.shields.io/badge/Redirect-3xx-blue) | Multiple Choices | The requested resource has multiple representations. |
| **301** | ![Redirect](https://img.shields.io/badge/Redirect-3xx-blue) | Moved Permanently | The resource has been permanently moved to a new URL. |
| **302** | ![Redirect](https://img.shields.io/badge/Redirect-3xx-blue) | Found | The resource has been temporarily moved to a different URL. |
| **303** | ![Redirect](https://img.shields.io/badge/Redirect-3xx-blue) | See Other | The client should use another URI to access the resource. |
| **304** | ![Redirect](https://img.shields.io/badge/Redirect-3xx-blue) | Not Modified | The cached version of the resource is still valid. |
| **307** | ![Redirect](https://img.shields.io/badge/Redirect-3xx-blue) | Temporary Redirect | The request should be repeated to another URL using the same method. |
| **308** | ![Redirect](https://img.shields.io/badge/Redirect-3xx-blue) | Permanent Redirect | The resource has been permanently redirected with the same method. |
| **400** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Bad Request | The server couldn’t understand the request due to invalid syntax. |
| **401** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Unauthorized | Authentication is required or failed. |
| **402** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Payment Required | Reserved for future use (e.g., digital payment systems). |
| **403** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Forbidden | The client doesn’t have permission to access the resource. |
| **404** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Not Found | The requested resource couldn’t be found on the server. |
| **405** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Method Not Allowed | The request method is not supported by the resource. |
| **406** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Not Acceptable | The requested format is not supported by the server. |
| **407** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Proxy Authentication Required | Authentication with a proxy server is required. |
| **408** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Request Timeout | The client took too long to send the request. |
| **409** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Conflict | The request conflicts with the current state of the resource. |
| **410** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Gone | The requested resource is no longer available. |
| **411** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Length Required | The request did not specify the content length. |
| **412** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Precondition Failed | One or more request conditions were not met. |
| **413** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Payload Too Large | The request payload exceeds server limits. |
| **414** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | URI Too Long | The request URI is longer than the server can process. |
| **415** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Unsupported Media Type | The request format is not supported. |
| **416** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Range Not Satisfiable | The requested range is invalid or unavailable. |
| **417** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Expectation Failed | The server cannot meet the expectations in the request header. |
| **418** | ![Fun](https://img.shields.io/badge/Fun-418-ff69b4) | I’m a Teapot | A playful code (from an April Fools' joke in HTTP 418). |
| **421** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Misdirected Request | The request was sent to the wrong server. |
| **422** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Unprocessable Entity | The request is well-formed but has semantic errors (e.g., validation). |
| **423** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Locked | The resource is locked. |
| **424** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Failed Dependency | The request failed due to a previous request error. |
| **425** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Too Early | The server is unwilling to process the request because it might be replayed. |
| **426** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Upgrade Required | The client should switch to a different protocol (e.g., HTTPS). |
| **428** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Precondition Required | The server requires request conditions (e.g., ETag). |
| **429** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Too Many Requests | The client sent too many requests in a short time (rate limiting). |
| **431** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Request Header Fields Too Large | Request headers are too large to process. |
| **451** | ![Client Error](https://img.shields.io/badge/Client%20Error-4xx-orange) | Unavailable For Legal Reasons | The resource is blocked due to legal restrictions. |
| **500** | ![Server Error](https://img.shields.io/badge/Server%20Error-5xx-red) | Internal Server Error | A generic error occurred on the server. |
| **501** | ![Server Error](https://img.shields.io/badge/Server%20Error-5xx-red) | Not Implemented | The server doesn’t support the requested functionality. |
| **502** | ![Server Error](https://img.shields.io/badge/Server%20Error-5xx-red) | Bad Gateway | The server received an invalid response from an upstream server. |
| **503** | ![Server Error](https://img.shields.io/badge/Server%20Error-5xx-red) | Service Unavailable | The server is temporarily overloaded or under maintenance. |
| **504** | ![Server Error](https://img.shields.io/badge/Server%20Error-5xx-red) | Gateway Timeout | The server didn’t receive a timely response from another server. |

---

## 💡 Notes

- **2xx** → Successful operations  
- **3xx** → Redirects or resource movement  
- **4xx** → Client-side issues (fix your request)  
- **5xx** → Server-side problems (backend/server down or misconfigured)

---

### 🧩 Example API Response

```json
{
  "status": 404,
  "message": "Resource not found"
}
