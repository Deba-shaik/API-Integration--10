# Topic 10 - Building Production-Ready API Integration Projects

## Overview
This n8n hands-on implementation demonstrates a five-minute scheduled API uptime check, routing of healthy and failed requests, a test downtime alert POST, and a separately authenticated webhook that responds with JSON. The work follows the supplied Topic 10 implementation guide.

## Workflow Architecture

**Configuration test branch:** Manual Trigger -> Load Config

**Uptime monitor:** Schedule Trigger (every 5 minutes) -> Ping API Endpoint (HTTP GET) -> Check API Status (statusCode = 200) -> API Healthy (True) / Send Downtime Alert (False). The Ping API Endpoint's Error output also connects to Send Downtime Alert.

**Authenticated webhook:** Secure Production Webhook (POST, Header Auth) -> Webhook Accepted (Edit Fields) -> Respond to Webhook (HTTP 200 JSON).

## Nodes and Configuration
| Node | Purpose / observed setup |
| --- | --- |
| Manual Trigger | Tests the configuration branch on demand. |
| Load Config | Outputs endpoint `https://reqres.in/api` and environment `development` as fixed values. |
| Schedule Trigger | Configured for five-minute intervals. |
| Ping API Endpoint | GET `https://jsonplaceholder.typicode.com/users/2`; includes response status; error output enabled. |
| Check API Status | Routes `statusCode == 200` to True. |
| API Healthy | No Operation node on True branch. |
| Send Downtime Alert | POST `https://postman-echo.com/post` with alert, endpoint, statusCode, errorMessage, timestamp. This is a test echo, not a delivered notification. |
| Secure Production Webhook | POST `/production-api-security`, Header Auth with secret credential. |
| Webhook Accepted | Sets `status=accepted`, `message=Authenticated webhook received`, `environment=development`. |
| Respond to Webhook | JSON response with HTTP 200, using `JSON.stringify($json)` in the Response Body expression. |

## Verified Tests
1. Load Config executed and produced one item with the expected fixed development values.
2. Five-minute Schedule Trigger executed manually.
3. Healthy API returned HTTP 200; Check API Status routed one item to API Healthy.
4. A failed ping generated an error item, and Send Downtime Alert successfully posted to Postman Echo. The observed test error was HTTP 404; the alert body used statusCode 0 because the error code was not parsed from the error message.
5. Postman rejected an unauthorized webhook request with HTTP 403 and `Authorization data is wrong!`.
6. Postman accepted the authorized POST and returned HTTP 200 with JSON: `status=accepted`, `message=Authenticated webhook received`, `environment=development`.
7. The final canvas screenshot shows the configuration, uptime, and secure webhook branches.

## Test Payload
```json
{
  "event": "api.healthcheck",
  "source": "postman",
  "environment": "development",
  "status": "testing"
}
```

## Operational and Security Notes
- The instance denied `$env` access, so Load Config uses fixed development values. The scheduled uptime branch is not linked to that configuration node; full development/production environment separation has not been implemented.
- Header Auth validates an API key; HMAC/signature verification was not demonstrated.
- A webhook API key was visible in a screenshot. Rotate it before any real deployment and do not include secrets in exported artifacts or evidence.
- Postman Echo receives test alert data only; it does not notify a person or service.
- The node version upgrade review, actual five-minute published execution, and production deployment were not evidenced in the screenshots.
- The workflow JSON was reported as exported, but was not uploaded for independent configuration review.

## Submission Artifacts
- Exported n8n workflow JSON (kept by the author).
- Topic_10_README.md (this file).
- Topic_10_Assessment_Evidence.pdf (screenshots and test explanations).
- Optional five-minute screen recording showing both success and failure tests.
