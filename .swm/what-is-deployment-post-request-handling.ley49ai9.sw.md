---
title: What is Deployment Post Request Handling
---
The DeploymentPostController is a class responsible for handling HTTP POST requests related to deployments. It contains methods to handle incoming requests, validate the deployment, and send responses back to the client. It also includes error handling for various scenarios such as unsupported media types, invalid deployments, and connection errors.

The DeploymentPostController uses a Proxy object to interact with the underlying system, a ProxyContext object to maintain the state of the current request, a CustomApplicationService to interact with custom applications, and a list of BaseFunction objects for request enhancement.

The handle method is the entry point for handling a deployment request. It validates the request, selects the appropriate deployment, checks access permissions, and then processes the request accordingly.

The sendRequest method is responsible for sending the request to the appropriate route. It checks if there is a next route available, builds the URI for the request, and then sends the request. If there is a failure in connecting to the route, it handles the error accordingly.

The handleRequestBody method is used to process the body of the request. It logs the received body, sets the request body in the context, and then processes the body. If there is an error in processing the body, it handles the error and sends a response back to the client.

The handleResponse method is used to handle the response from the origin. It sets the response body in the context, calculates the cost of the model if applicable, and then logs the response. It also saves the context to the log store.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentPostController.java" line="54">

---

# DeploymentPostController Class

The DeploymentPostController class is defined here. It has four instance variables: a Proxy object, a ProxyContext object, a CustomApplicationService, and a list of BaseFunction objects. These are initialized in the constructor.

```java
@Slf4j
public class DeploymentPostController {

    private static final Set<Integer> DEFAULT_RETRIABLE_HTTP_CODES = Set.of(HttpStatus.TOO_MANY_REQUESTS.getCode(),
            HttpStatus.BAD_GATEWAY.getCode(), HttpStatus.GATEWAY_TIMEOUT.getCode(),
            HttpStatus.SERVICE_UNAVAILABLE.getCode());

    private final Proxy proxy;
    private final ProxyContext context;
    private final CustomApplicationService applicationService;
    private final List<BaseFunction<ObjectNode>> enhancementFunctions;

    public DeploymentPostController(Proxy proxy, ProxyContext context) {
        this.proxy = proxy;
        this.context = context;
        this.applicationService = proxy.getCustomApplicationService();
        this.enhancementFunctions = List.of(new CollectAttachmentsFn(proxy, context),
                new ApplyDefaultDeploymentSettingsFn(proxy, context),
                new EnhanceAssistantRequestFn(proxy, context),
                new EnhanceModelRequestFn(proxy, context));
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentPostController.java" line="76">

---

# handle Method

The handle method is the entry point for handling a deployment request. It validates the request, selects the appropriate deployment, checks access permissions, and then processes the request accordingly.

```java
    public Future<?> handle(String deploymentId, String deploymentApi) {
        String contentType = context.getRequest().getHeader(HttpHeaders.CONTENT_TYPE);
        if (!StringUtils.containsIgnoreCase(contentType, Proxy.HEADER_CONTENT_TYPE_APPLICATION_JSON)) {
            return respond(HttpStatus.UNSUPPORTED_MEDIA_TYPE, "Only application/json is supported");
        }

        Deployment deployment = context.getConfig().selectDeployment(deploymentId);
        boolean isValidDeployment = isValidDeploymentApi(deployment, deploymentApi);

        if (!isValidDeployment) {
            log.warn("Deployment {}/{} is not valid", deploymentId, deploymentApi);
            return respond(HttpStatus.NOT_FOUND, "Deployment is not found");
        }

        Future<Deployment> deploymentFuture;
        if (deployment != null) {
            if (!isBaseAssistant(deployment) && !DeploymentController.hasAccess(context, deployment)) {
                log.error("Forbidden deployment {}. Key: {}. User sub: {}", deploymentId, context.getProject(), context.getUserSub());
                return respond(HttpStatus.FORBIDDEN, "Forbidden deployment: " + deploymentId);
            }
            deploymentFuture = Future.succeededFuture(deployment);
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentPostController.java" line="201">

---

# sendRequest Method

The sendRequest method is responsible for sending the request to the appropriate route. It checks if there is a next route available, builds the URI for the request, and then sends the request. If there is a failure in connecting to the route, it handles the error accordingly.

```java
    private void sendRequest() {
        UpstreamRoute route = context.getUpstreamRoute();
        HttpServerRequest request = context.getRequest();

        if (!route.hasNext()) {
            log.error("No route. Trace: {}. Span: {}. Key: {}. Deployment: {}. User sub: {}",
                    context.getTraceId(), context.getSpanId(),
                    context.getProject(), context.getDeployment().getName(), context.getUserSub());

            respond(HttpStatus.BAD_GATEWAY, "No route");
            return;
        }

        Upstream upstream = route.next();
        Objects.requireNonNull(upstream);

        String uri = buildUri(context);
        RequestOptions options = new RequestOptions()
                .setAbsoluteURI(uri)
                .setMethod(request.method());

```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentPostController.java" line="227">

---

# handleRequestBody Method

The handleRequestBody method is used to process the body of the request. It logs the received body, sets the request body in the context, and then processes the body. If there is an error in processing the body, it handles the error and sends a response back to the client.

```java
    @VisibleForTesting
    void handleRequestBody(Buffer requestBody) {
        Deployment deployment = context.getDeployment();
        log.info("Received body from client. Trace: {}. Span: {}. Key: {}. Deployment: {}. Length: {}",
                context.getTraceId(), context.getSpanId(),
                context.getProject(), deployment.getName(), requestBody.length());

        context.setRequestBody(requestBody);
        context.setRequestBodyTimestamp(System.currentTimeMillis());

        try (InputStream stream = new ByteBufInputStream(requestBody.getByteBuf())) {
            ObjectNode tree = (ObjectNode) ProxyUtil.MAPPER.readTree(stream);
            Throwable error = ProxyUtil.processChain(tree, enhancementFunctions);
            if (error != null) {
                finalizeRequest();
                return;
            }
        } catch (IOException e) {
            respond(HttpStatus.BAD_REQUEST);
            log.warn("Can't parse JSON request body. Trace: {}. Span: {}. Error:",
                    context.getTraceId(), context.getSpanId(), e);
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentPostController.java" line="339">

---

# handleResponse Method

The handleResponse method is used to handle the response from the origin. It sets the response body in the context, calculates the cost of the model if applicable, and then logs the response. It also saves the context to the log store.

```java
    @VisibleForTesting
    void handleResponse() {
        Buffer responseBody = context.getResponseStream().getContent();
        context.setResponseBody(responseBody);
        context.setResponseBodyTimestamp(System.currentTimeMillis());
        Future<TokenUsage> tokenUsageFuture = Future.succeededFuture();
        if (context.getDeployment() instanceof Model model) {
            if (context.getResponse().getStatusCode() == HttpStatus.OK.getCode()) {
                TokenUsage tokenUsage = TokenUsageParser.parse(responseBody);
                if (tokenUsage == null) {
                    Pricing pricing = model.getPricing();
                    if (pricing == null || "token".equals(pricing.getUnit())) {
                        log.warn("Can't find token usage. Trace: {}. Span: {}. Key: {}. Deployment: {}. Endpoint: {}. Upstream: {}. Status: {}. Length: {}",
                                context.getTraceId(), context.getSpanId(),
                                context.getProject(), context.getDeployment().getName(),
                                context.getDeployment().getEndpoint(),
                                context.getUpstreamRoute().get().getEndpoint(),
                                context.getResponse().getStatusCode(),
                                context.getResponseBody().length());
                    }
                    tokenUsage = new TokenUsage();
```

---

</SwmSnippet>

# DeploymentPostController Functions

This section will provide an overview of the main functions in the DeploymentPostController class.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentPostController.java" line="76">

---

## Handle Function

The handle function is the entry point for handling a deployment request. It validates the request, selects the appropriate deployment, checks access permissions, and then processes the request accordingly.

```java
    public Future<?> handle(String deploymentId, String deploymentApi) {
        String contentType = context.getRequest().getHeader(HttpHeaders.CONTENT_TYPE);
        if (!StringUtils.containsIgnoreCase(contentType, Proxy.HEADER_CONTENT_TYPE_APPLICATION_JSON)) {
            return respond(HttpStatus.UNSUPPORTED_MEDIA_TYPE, "Only application/json is supported");
        }

        Deployment deployment = context.getConfig().selectDeployment(deploymentId);
        boolean isValidDeployment = isValidDeploymentApi(deployment, deploymentApi);

        if (!isValidDeployment) {
            log.warn("Deployment {}/{} is not valid", deploymentId, deploymentApi);
            return respond(HttpStatus.NOT_FOUND, "Deployment is not found");
        }

        Future<Deployment> deploymentFuture;
        if (deployment != null) {
            if (!isBaseAssistant(deployment) && !DeploymentController.hasAccess(context, deployment)) {
                log.error("Forbidden deployment {}. Key: {}. User sub: {}", deploymentId, context.getProject(), context.getUserSub());
                return respond(HttpStatus.FORBIDDEN, "Forbidden deployment: " + deploymentId);
            }
            deploymentFuture = Future.succeededFuture(deployment);
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentPostController.java" line="201">

---

## SendRequest Function

The sendRequest function is responsible for sending the request to the appropriate route. It checks if there is a next route available, builds the URI for the request, and then sends the request. If there is a failure in connecting to the route, it handles the error accordingly.

```java
    private void sendRequest() {
        UpstreamRoute route = context.getUpstreamRoute();
        HttpServerRequest request = context.getRequest();

        if (!route.hasNext()) {
            log.error("No route. Trace: {}. Span: {}. Key: {}. Deployment: {}. User sub: {}",
                    context.getTraceId(), context.getSpanId(),
                    context.getProject(), context.getDeployment().getName(), context.getUserSub());

            respond(HttpStatus.BAD_GATEWAY, "No route");
            return;
        }

        Upstream upstream = route.next();
        Objects.requireNonNull(upstream);

        String uri = buildUri(context);
        RequestOptions options = new RequestOptions()
                .setAbsoluteURI(uri)
                .setMethod(request.method());

```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentPostController.java" line="228">

---

## HandleRequestBody Function

The handleRequestBody function is used to process the body of the request. It logs the received body, sets the request body in the context, and then processes the body. If there is an error in processing the body, it handles the error and sends a response back to the client.

```java
    void handleRequestBody(Buffer requestBody) {
        Deployment deployment = context.getDeployment();
        log.info("Received body from client. Trace: {}. Span: {}. Key: {}. Deployment: {}. Length: {}",
                context.getTraceId(), context.getSpanId(),
                context.getProject(), deployment.getName(), requestBody.length());

        context.setRequestBody(requestBody);
        context.setRequestBodyTimestamp(System.currentTimeMillis());

        try (InputStream stream = new ByteBufInputStream(requestBody.getByteBuf())) {
            ObjectNode tree = (ObjectNode) ProxyUtil.MAPPER.readTree(stream);
            Throwable error = ProxyUtil.processChain(tree, enhancementFunctions);
            if (error != null) {
                finalizeRequest();
                return;
            }
        } catch (IOException e) {
            respond(HttpStatus.BAD_REQUEST);
            log.warn("Can't parse JSON request body. Trace: {}. Span: {}. Error:",
                    context.getTraceId(), context.getSpanId(), e);
            return;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentPostController.java" line="339">

---

## HandleResponse Function

The handleResponse function is used to handle the response from the origin. It sets the response body in the context, calculates the cost of the model if applicable, and then logs the response. It also saves the context to the log store.

```java
    @VisibleForTesting
    void handleResponse() {
        Buffer responseBody = context.getResponseStream().getContent();
        context.setResponseBody(responseBody);
        context.setResponseBodyTimestamp(System.currentTimeMillis());
        Future<TokenUsage> tokenUsageFuture = Future.succeededFuture();
        if (context.getDeployment() instanceof Model model) {
            if (context.getResponse().getStatusCode() == HttpStatus.OK.getCode()) {
                TokenUsage tokenUsage = TokenUsageParser.parse(responseBody);
                if (tokenUsage == null) {
                    Pricing pricing = model.getPricing();
                    if (pricing == null || "token".equals(pricing.getUnit())) {
                        log.warn("Can't find token usage. Trace: {}. Span: {}. Key: {}. Deployment: {}. Endpoint: {}. Upstream: {}. Status: {}. Length: {}",
                                context.getTraceId(), context.getSpanId(),
                                context.getProject(), context.getDeployment().getName(),
                                context.getDeployment().getEndpoint(),
                                context.getUpstreamRoute().get().getEndpoint(),
                                context.getResponse().getStatusCode(),
                                context.getResponseBody().length());
                    }
                    tokenUsage = new TokenUsage();
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
