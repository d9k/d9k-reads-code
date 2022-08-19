# Superset: async chart data

- doc: Async Queries via Celery
  - https://superset.apache.org/docs/installation/async-queries-celery/

- Superset version: `1.4.0rc2`

- SuperSet Swagger:
  - https://superset.apache.org/docs/api/

## Async для docker compose

Судя по исходному коду Superset, его сервисы `superset-worker` и `superset-worker-beat` и так по-умолчанию стартуют с поддержкой асинхронности (команды
`celery --app=superset.tasks.celery_app:app worker -Ofair -l INFO`
и
`celery --app=superset.tasks.celery_app:app beat --pidfile /tmp/celerybeat.pid -l INFO -s "${SUPERSET_HOME}"/celerybeat-schedule`
)

см.:
- https://github.com/apache/superset/blob/1.4.0rc2/docker-compose-non-dev.yml
- https://github.com/apache/superset/blob/1.4.0rc2/docker/docker-bootstrap.sh

## openapi.json:

https://github.com/apache/superset/blob/1.4.0rc2/docs/src/resources/openapi.json

`ChartDataAsyncResponseSchema`,

`allow_run_async` in some queries:

```
    "/chart/data": {
      "post": {
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "$ref": "#/components/schemas/ChartDataQueryContextSchema"
              }
            }
          },
        },
        "responses": {
          "202": {
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/ChartDataAsyncResponseSchema"
                }
              }
            },
            "description": "Async job details"
          },
```

## search: AsyncResponse

```
superset-frontend/src/middleware/asyncEvent.ts
```

```
export const waitForAsyncData = async (asyncResponse: AsyncEvent) =>
```

## search: 202

### superset/charts/api.py

```
def _run_async(self, command: ChartDataCommand) -> Response:
    # First, look for the chart query results in the cache.
    command.validate_async_request(request)
    result = command.run_async(g.user.get_id())
```

### superset-frontend/src/chart/chartAction.js

```
const chartDataRequestCaught = chartDataRequest
  .then(({ response, json }) => {
    if (isFeatureEnabled(FeatureFlag.GLOBAL_ASYNC_QUERIES)) {
    switch (response.status) {
      case 200:
        if (shouldUseLegacyApi(formData)) {
          return waitForAsyncData(result[0]);
        }
        return waitForAsyncData(result);
```

## search: GLOBAL_ASYNC_QUERIES

### docs/installation.rst:

- ``GLOBAL_ASYNC_QUERIES`` (feature flag) - enable or disable async query operation
- ``GLOBAL_ASYNC_QUERIES_REDIS_CONFIG`` - Redis connection info
- ``GLOBAL_ASYNC_QUERIES_REDIS_STREAM_PREFIX`` - the prefix used with Redis Streams
- ``GLOBAL_ASYNC_QUERIES_REDIS_STREAM_LIMIT`` - the maximum number of events for each user-specific event stream (FIFO eviction)
- ``GLOBAL_ASYNC_QUERIES_REDIS_STREAM_LIMIT_FIREHOSE`` - the maximum number of events for all users (FIFO eviction)
- ``GLOBAL_ASYNC_QUERIES_JWT_COOKIE_NAME`` - the async query feature uses a `JWT <https://tools.ietf.org/html/rfc7519>`_ cookie for authentication, this setting is the cookie's name
- ``GLOBAL_ASYNC_QUERIES_JWT_COOKIE_SECURE`` - JWT cookie secure option
- ``GLOBAL_ASYNC_QUERIES_JWT_COOKIE_DOMAIN`` - JWT cookie domain option (`see docs for set_cookie <https://tedboy.github.io/flask/interface_api.response_object.html#flask.Response.set_cookie>`
- ``GLOBAL_ASYNC_QUERIES_JWT_SECRET`` - JWT's use a secret key to sign and validate the contents. This value should be at least 32 bytes and have sufficient randomness for proper security
- ``GLOBAL_ASYNC_QUERIES_TRANSPORT`` - available options: "polling" (HTTP, default), "ws" (WebSocket, requires running superset-websocket server)
- ``GLOBAL_ASYNC_QUERIES_POLLING_DELAY`` - the time (in ms) between polling requests
