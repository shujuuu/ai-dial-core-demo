---
title: Rate Limiting Overview
---
Rate Limiting in ai-dial-core-demo refers to the process of controlling the frequency of user requests to the API to prevent overuse. It is implemented using the `RateLimiter` class, which provides methods to increase, limit, and get statistics about the rate limit.

The `RateLimiter` class uses the `Vertx` instance and the `ResourceService` to manage the rate limit. It has a `DEFAULT_LIMIT` constant that sets the default limit for the rate limiter.

The `increase` method in the `RateLimiter` class is used to increase the token usage. If the `resourceService` is not available or the total tokens are less than or equal to zero, it skips the limit check.

The `limit` method in the `RateLimiter` class is used to check if the request exceeds the limit. If the limit is not positive or not found, it returns an access denied message.

The `getLimitStats` method in the `RateLimiter` class is used to get the statistics of the limit. It returns the limit statistics for a specific deployment.

The `RequestRateLimit` class is used to check the rate limit for requests. It has two `RateBucket` instances for hour and day. The `check` method in this class checks if the request exceeds the hourly or daily limit.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/limiter/RateLimiter.java" line="25">

---

# RateLimiter Class

The `RateLimiter` class is the main class responsible for implementing rate limiting. It uses the `Vertx` instance and the `ResourceService` to manage the rate limit. It also has a `DEFAULT_LIMIT` constant that sets the default limit for the rate limiter.

```java
@Slf4j
@RequiredArgsConstructor
public class RateLimiter {

    private static final Limit DEFAULT_LIMIT = new Limit();
    private static final String DEFAULT_USER_ROLE = "default";

    private final Vertx vertx;

    private final ResourceService resourceService;

```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/limiter/RateLimiter.java" line="36">

---

# Increase Method

The `increase` method in the `RateLimiter` class is used to increase the token usage. If the `resourceService` is not available or the total tokens are less than or equal to zero, it skips the limit check.

```java
    public Future<Void> increase(ProxyContext context) {
        try {
            // skip checking limits if redis is not available
            if (resourceService == null) {
                return Future.succeededFuture();
            }

            TokenUsage usage = context.getTokenUsage();

            if (usage == null || usage.getTotalTokens() <= 0) {
                return Future.succeededFuture();
            }

            String tokensPath = getPathToTokens(context.getDeployment().getName());
            ResourceDescription resourceDescription = getResourceDescription(context, tokensPath);
            return vertx.executeBlocking(() -> updateTokenLimit(resourceDescription, usage.getTotalTokens()), false);
        } catch (Throwable e) {
            return Future.failedFuture(e);
        }
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/limiter/RateLimiter.java" line="57">

---

# Limit Method

The `limit` method in the `RateLimiter` class is used to check if the request exceeds the limit. If the limit is not positive or not found, it returns an access denied message.

```java
    public Future<RateLimitResult> limit(ProxyContext context) {
        try {
            // skip checking limits if redis is not available
            if (resourceService == null) {
                return Future.succeededFuture(RateLimitResult.SUCCESS);
            }
            Key key = context.getKey();
            String deploymentName = context.getDeployment().getName();
            Limit limit;
            if (key == null) {
                limit = getLimitByUser(context, deploymentName);
            } else {
                limit = getLimitByApiKey(context, deploymentName);
            }

            if (limit == null || !limit.isPositive()) {
                if (limit == null) {
                    log.warn("Limit is not found for deployment: {}", deploymentName);
                } else {
                    log.warn("Limit must be positive for deployment: {}", deploymentName);
                }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/limiter/RateLimiter.java" line="87">

---

# GetLimitStats Method

The `getLimitStats` method in the `RateLimiter` class is used to get the statistics of the limit. It returns the limit statistics for a specific deployment.

```java
    public Future<LimitStats> getLimitStats(String deploymentName, ProxyContext context) {
        try {
            // skip checking limits if redis is not available
            if (resourceService == null) {
                return Future.succeededFuture();
            }
            Key key = context.getKey();
            Limit limit;
            if (key == null) {
                limit = getLimitByUser(context, deploymentName);
            } else {
                limit = getLimitByApiKey(context, deploymentName);
            }
            if (limit == null) {
                log.warn("Limit is not found. Trace: {}. Span: {}. Key: {}. User sub: {}. Deployment: {}",
                        context.getTraceId(), context.getSpanId(), key == null ? null : key.getProject(),
                        context.getUserSub(), deploymentName);
                return Future.succeededFuture();
            }
            return vertx.executeBlocking(() -> getLimitStats(context, limit, deploymentName), false);
        } catch (Throwable e) {
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/limiter/RequestRateLimit.java" line="9">

---

# RequestRateLimit Class

The `RequestRateLimit` class is used to check the rate limit for requests. It has two `RateBucket` instances for hour and day. The `check` method in this class checks if the request exceeds the hourly or daily limit.

```java
public class RequestRateLimit {
    private final RateBucket hour = new RateBucket(RateWindow.HOUR);
    private final RateBucket day = new RateBucket(RateWindow.DAY);

    public RateLimitResult check(long timestamp, Limit limit, long count) {
        long hourTotal = hour.update(timestamp);
        long dayTotal = day.update(timestamp);

        boolean result = hourTotal >= limit.getRequestHour() || dayTotal >= limit.getRequestDay();
        if (result) {
            String errorMsg = String.format("""
                            Hit request rate limit:
                             - hour limit: %d / %d requests
                             - day limit: %d / %d requests
                            """,
                    hourTotal, limit.getRequestHour(), dayTotal, limit.getRequestDay());
            return new RateLimitResult(HttpStatus.TOO_MANY_REQUESTS, errorMsg);
        } else {
            hour.add(timestamp, count);
            day.add(timestamp, count);
            return RateLimitResult.SUCCESS;
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
