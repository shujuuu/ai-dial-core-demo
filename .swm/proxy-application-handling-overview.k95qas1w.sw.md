---
title: Proxy Application Handling Overview
---
Proxy Application Handling refers to the way the ai-dial-core-demo manages and routes HTTP requests to different applications. The `ApplicationController` class is responsible for handling application-specific requests. It uses the `ProxyContext` and `Proxy` objects to manage the context of the request and the proxy server respectively.

The `handleProxyRequest` method in `DeploymentFeatureController` and `RouteController` classes is responsible for handling incoming proxy requests. It copies headers from the original request to the proxy request, sets the content length, and sends the request to the destination.

The `ControllerSelector` class is used to select the appropriate controller based on the HTTP method of the incoming request. It uses the `select` method to determine the correct controller to handle the request.

The `hasAccess` method in `DeploymentController` class is used to check if the current context has access to a specific deployment. It checks both the access limits and user roles to determine access.

The `DeploymentPostController` class handles POST requests to deployments. It validates the deployment and handles rate limiting. If the rate limit is hit, it logs the event and responds with an error. If the rate limit is not hit, it processes the request and sends it to the destination.

The `RouteController` class handles requests that do not match any specific application or deployment. It selects the appropriate route based on the request and forwards the request to the destination.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ApplicationController.java" line="21">

---

# ApplicationController Class

The ApplicationController class is responsible for handling application-specific requests. It uses the ProxyContext and Proxy objects to manage the context of the request and the proxy server respectively.

```java
public class ApplicationController {

    private final ProxyContext context;
    private final Vertx vertx;
    private final CustomApplicationService customApplicationService;
    private final boolean includeCustomApplications;

    public ApplicationController(ProxyContext context, Proxy proxy) {
        this.context = context;
        this.vertx = proxy.getVertx();
        this.customApplicationService = proxy.getCustomApplicationService();
        this.includeCustomApplications = customApplicationService.includeCustomApplications();
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentFeatureController.java" line="105">

---

# handleProxyRequest Method

The handleProxyRequest method in DeploymentFeatureController class is responsible for handling incoming proxy requests. It copies headers from the original request to the proxy request, sets the content length, and sends the request to the destination.

```java
    /**
     * Called when proxy connected to the origin.
     */
    private void handleProxyRequest(HttpClientRequest proxyRequest) {
        log.info("Connected to origin: {}", proxyRequest.connection().remoteAddress());

        HttpServerRequest request = context.getRequest();
        context.setProxyRequest(proxyRequest);

        ProxyUtil.copyHeaders(request.headers(), proxyRequest.headers());

        Buffer proxyRequestBody = context.getRequestBody();
        proxyRequest.putHeader(HttpHeaders.CONTENT_LENGTH, Integer.toString(proxyRequestBody.length()));

        proxyRequest.send(proxyRequestBody)
                .onSuccess(this::handleProxyResponse)
                .onFailure(this::handleProxyRequestError);
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="63">

---

# ControllerSelector Class

The ControllerSelector class is used to select the appropriate controller based on the HTTP method of the incoming request. It uses the select method to determine the correct controller to handle the request.

```java
    public Controller select(Proxy proxy, ProxyContext context) {
        String path = context.getRequest().path();
        HttpMethod method = context.getRequest().method();
        Controller controller = null;

        if (method == HttpMethod.GET) {
            controller = selectGet(proxy, context, path);
        } else if (method == HttpMethod.POST) {
            controller = selectPost(proxy, context, path);
        } else if (method == HttpMethod.DELETE) {
            controller = selectDelete(proxy, context, path);
        } else if (method == HttpMethod.PUT) {
            controller = selectPut(proxy, context, path);
        }

        return (controller == null) ? new RouteController(proxy, context) : controller;
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentController.java" line="62">

---

# hasAccess Method

The hasAccess method in DeploymentController class is used to check if the current context has access to a specific deployment. It checks both the access limits and user roles to determine access.

```java
    public static boolean hasAccess(ProxyContext context, Deployment deployment) {
        return hasAssessByLimits(context, deployment) && hasAccessByUserRoles(context, deployment);
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentPostController.java" line="91">

---

# DeploymentPostController Class

The DeploymentPostController class handles POST requests to deployments. It validates the deployment and handles rate limiting. If the rate limit is hit, it logs the event and responds with an error. If the rate limit is not hit, it processes the request and sends it to the destination.

```java
        if (deployment != null) {
            if (!isBaseAssistant(deployment) && !DeploymentController.hasAccess(context, deployment)) {
                log.error("Forbidden deployment {}. Key: {}. User sub: {}", deploymentId, context.getProject(), context.getUserSub());
```

---

</SwmSnippet>

# Endpoint Handling in DeploymentPostController and RouteController

Understanding the handle methods in DeploymentPostController and RouteController

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentPostController.java" line="39">

---

## DeploymentPostController

The `handle` method in the `DeploymentPostController` class is responsible for handling POST requests to deployments. It first validates the deployment and checks if it has access. If the rate limit is hit, it logs the event and responds with an error. If the rate limit is not hit, it processes the request and sends it to the destination.

```java
import io.vertx.core.http.HttpHeaders;
import io.vertx.core.http.HttpServerRequest;
import io.vertx.core.http.HttpServerResponse;
import io.vertx.core.http.RequestOptions;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;

import java.io.IOException;
import java.io.InputStream;
import java.math.BigDecimal;
import java.util.List;
import java.util.Objects;
import java.util.Set;

@Slf4j
public class DeploymentPostController {

    private static final Set<Integer> DEFAULT_RETRIABLE_HTTP_CODES = Set.of(HttpStatus.TOO_MANY_REQUESTS.getCode(),
            HttpStatus.BAD_GATEWAY.getCode(), HttpStatus.GATEWAY_TIMEOUT.getCode(),
            HttpStatus.SERVICE_UNAVAILABLE.getCode());
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/RouteController.java" line="39">

---

## RouteController

The `handle` method in the `RouteController` class is responsible for handling requests that do not match any specific application or deployment. It selects the appropriate route based on the request and forwards the request to the destination.

```java
    @Override
    public Future<?> handle() {
        Route route = selectRoute();
        if (route == null) {
            log.warn("RouteController can't find a route to proceed the request: {}", getRequestUri());
            context.respond(HttpStatus.BAD_GATEWAY, "No route");
            return Future.succeededFuture();
        }

        Route.Response response = route.getResponse();
        if (response == null) {
            UpstreamProvider upstreamProvider = new RouteEndpointProvider(route);
            UpstreamRoute upstreamRoute = proxy.getUpstreamBalancer().balance(upstreamProvider);

            if (!upstreamRoute.hasNext()) {
                log.warn("RouteController can't find a upstream route to proceed the request: {}", getRequestUri());
                context.respond(HttpStatus.BAD_GATEWAY, "No route");
                return Future.succeededFuture();
            }

            context.setUpstreamRoute(upstreamRoute);
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
