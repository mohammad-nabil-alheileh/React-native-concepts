# HTTP status codes

Every HTTP response includes a three-digit number that tells you, before you even read the body, whether the request worked and, if not, whose fault it was. The first digit gives the category.

| Range | Meaning | Who is at fault |
| --- | --- | --- |
| 2xx | Success | Nobody |
| 3xx | Redirect | The resource moved |
| 4xx | Client error | Your app sent something wrong |
| 5xx | Server error | The server failed |

## The codes you will actually meet

### Success

| Code | Name | Meaning |
| --- | --- | --- |
| 200 | OK | Request worked, the body has the data |
| 201 | Created | A POST created something |
| 204 | No Content | Worked, but there is intentionally no body |

### Client errors (4xx)

| Code | Name | Meaning |
| --- | --- | --- |
| 400 | Bad Request | Malformed or invalid data (a missing field, bad JSON) |
| 401 | Unauthorized | You are not authenticated |
| 403 | Forbidden | You are authenticated but not allowed to do this |
| 404 | Not Found | The resource does not exist |
| 409 | Conflict | Clashes with existing data (for example the email is already registered) |
| 422 | Unprocessable Entity | The data is well formed but fails validation rules |
| 429 | Too Many Requests | You are being rate limited |

### Server errors (5xx)

| Code | Name | Meaning |
| --- | --- | --- |
| 500 | Internal Server Error | The server crashed |
| 502 / 503 / 504 | Bad Gateway / Service Unavailable / Gateway Timeout | The server is down or overloaded |

## How to read them in Axios

Because Axios throws on anything outside 2xx, the status lives on `error.response.status`:

```js
try {
  const { data, status } = await api.post("/plants", { name: "Aloe" });
  console.log(status); // 201
} catch (error) {
  if (error.response) {
    // The server answered with a 4xx or 5xx
    const status = error.response.status;

    if (status === 400 || status === 422) console.log("Fix your input");
    else if (status === 401) console.log("Log in again");
    else if (status === 403) console.log("You can't do this");
    else if (status === 404) console.log("Not found");
    else if (status >= 500) console.log("Server problem, try later");
  } else if (error.request) {
    // The request was sent but no response came back (offline, timeout)
    console.log("Network problem");
  } else {
    // Something went wrong while building the request
    console.log(error.message);
  }
}
```

Client errors usually should not be retried (the same bad request will fail again). Server errors and 429 are the ones that may succeed on retry.
