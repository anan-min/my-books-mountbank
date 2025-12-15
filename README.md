# Mountebank Payment API Mock

This project provides a mountebank configuration to mock payment session creation and payment processing APIs for testing purposes.

## Project Structure

```
my-books-mountbank/
├── imposters.ejs                 # Main imposter configuration
├── test.http                     # HTTP test requests
└── payment/
    ├── session/
    │   ├── success/
    │   │   ├── session_success_predicate.ejs
    │   │   └── session_success_response.ejs
    │   └── error/
    │       ├── session_error_predicate.ejs
    │       └── session_error_response.ejs
    └── pay/
        ├── success/
        │   ├── pay_success_predicate.ejs
        │   └── pay_success_response.ejs
        ├── fail/
        │   ├── pay_fail_predicate.ejs
        │   └── pay_fail_response.ejs
        └── error/
            ├── pay_error_predicate.ejs
            └── pay_error_response.ejs
```

## Prerequisites

- [Node.js](https://nodejs.org/) installed
- [Mountebank](http://www.mbtest.org/) installed globally:
  ```sh
  npm install -g mountebank
  ```

## Getting Started

### 1. Start Mountebank

```sh
mb --configfile imposters.ejs --allowInjection
```

The mock server will start on port **6969**.

### 2. Stop Mountebank

```sh
mb stop
```

### 3. Restart Mountebank

```sh
mb restart --configfile imposters.ejs --allowInjection
```

## API Endpoints

### Payment Session API

#### `POST /payments/session`

Creates a new payment session.

**Success Response (200):**
```json
{
  "sessionId": "cs_test_<uuid>",
  "url": "https://checkout.stripe.com/pay/cs_test_mock"
}
```

**Error Response (500):**
Add header `X-Simulate-Error: true`
```json
{
  "error": {
    "type": "api_error",
    "message": "An error occurred while creating the session. Please try again later."
  }
}
```

---

### Payment API

#### `POST /payments/pay?sessionId=<sessionId>`

Processes a payment with the given session ID.

**Success Response (200):**
When `sessionId` ends with letters `a-f`:
```json
{
  "sessionId": "cs_test_...",
  "status": "success",
  "message": "Payment successful",
  "url": "https://checkout.stripe.com/pay/cs_test_..."
}
```

**Failure Response (402):**
When `sessionId` ends with numbers `0-9`:
```json
{
  "sessionId": "cs_test_...",
  "status": "failure",
  "message": "Payment failed",
  "url": "https://checkout.stripe.com/pay/cs_test_..."
}
```

**Error Response (500):**
Add header `X-Simulate-Error: true`
```json
{
  "error": {
    "type": "api_error",
    "message": "An error occurred while user pay. Please try again later."
  }
}
```

## Testing

Use the `test.http` file with VS Code REST Client extension or any HTTP client.

### Test Cases

1. **Session Creation Success**: `POST /payments/session`
2. **Session Creation Error**: `POST /payments/session` with header `X-Simulate-Error: true`
3. **Payment Success**: `POST /payments/pay?sessionId=cs_test_...a` (ends with letter)
4. **Payment Failure**: `POST /payments/pay?sessionId=cs_test_...4` (ends with number)
5. **Payment Error**: `POST /payments/pay` with header `X-Simulate-Error: true`

## How It Works

- **Session Success**: Returns a dynamically generated UUID-based session ID
- **Payment Success/Fail**: Determined by the last character of the `sessionId`:
  - Ends with `a-f` → Success (200)
  - Ends with `0-9` → Failure (402)
- **Error Simulation**: Both endpoints check for the `X-Simulate-Error` header to return 500 errors

## Configuration

The main configuration is in `imposters.ejs`, which includes:
- Port: `6969`
- Protocol: `http`
- Stubs: Defined in separate predicate and response files for maintainability

## Notes

- The `--allowInjection` flag is required because responses use JavaScript injection for dynamic content
- Session IDs are randomly generated with UUID format
- All responses use `Content-Type: application/json`
- Stub order matters: error stubs are checked first, then success/fail stubs

## Troubleshooting

### JSON Parse Errors
- Ensure all `inject` functions are on a single line without line breaks
- Check for missing commas between stubs in `imposters.ejs`

### Multiple Stubs on Same Route
- Use the `and` operator to combine predicates when differentiating stubs
- Order stubs from most specific to least specific

### Port Already in Use
If you see "port already in use" errors:
```sh
mb stop
mb restart --configfile imposters.ejs --allowInjection
```
