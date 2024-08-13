---
title: What is ApplicationController Proxy Handling
---
Proxy Application Handling refers to the process of managing requests and responses between the client and the server in the ApplicationController. The ApplicationController class uses a ProxyContext and a Proxy to initialize its state. The ProxyContext contains information about the current request, while the Proxy provides access to the Vertx instance and the CustomApplicationService.

The ApplicationController has a method called getApplication which is used to retrieve a specific application based on its ID. This method is invoked when a GET request is made to the '/application' endpoint with a specific application ID.

The ControllerSelector class plays a crucial role in handling requests. It has a method called select which determines the appropriate controller to handle a request based on the HTTP method and the request path. For instance, if the HTTP method is GET, the selectGet method is invoked to determine the appropriate controller.

The selectGet method matches the request path against various patterns to determine the appropriate controller. If the request path matches the '/application' pattern, an instance of ApplicationController is created and the getApplication method is invoked.

The handleProxyRequest method in the RouteController and DeploymentFeatureController classes is responsible for handling requests that are to be proxied to the origin server. This method copies headers from the original request to the proxy request, sets the content length of the proxy request, and sends the request to the origin server.

The hasAccess method in the DeploymentController class is used to check if a client has access to a specific deployment. This method checks if the client has access based on the limits set in the client's role and the user roles associated with the deployment.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ApplicationController.java" line="21">

---

# ApplicationController

The ApplicationController class uses a ProxyContext and a Proxy to initialize its state. The ProxyContext contains information about the current request, while the Proxy provides access to the Vertx instance and the CustomApplicationService.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ApplicationController.java" line="35">

---

# getApplication Method

The getApplication method in the ApplicationController is used to retrieve a specific application based on its ID. This method is invoked when a GET request is made to the '/application' endpoint with a specific application ID.

```java
    public Future<?> getApplication(String applicationId) {
        Config config = context.getConfig();
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="63">

---

# ControllerSelector

The ControllerSelector class has a method called select which determines the appropriate controller to handle a request based on the HTTP method and the request path. For instance, if the HTTP method is GET, the selectGet method is invoked to determine the appropriate controller.

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
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="81">

---

# selectGet Method

The selectGet method in the ControllerSelector class matches the request path against various patterns to determine the appropriate controller. If the request path matches the '/application' pattern, an instance of ApplicationController is created and the getApplication method is invoked.

```java
    private static Controller selectGet(Proxy proxy, ProxyContext context, String path) {
        Matcher match;

        match = match(PATTERN_DEPLOYMENT, path);
        if (match != null) {
            DeploymentController controller = new DeploymentController(context);
            String deploymentId = UrlUtil.decodePath(match.group(1));
            return () -> controller.getDeployment(deploymentId);
        }

        match = match(PATTERN_DEPLOYMENTS, path);
        if (match != null) {
            DeploymentController controller = new DeploymentController(context);
            return controller::getDeployments;
        }

        match = match(PATTERN_MODEL, path);
        if (match != null) {
            ModelController controller = new ModelController(context);
            String modelId = UrlUtil.decodePath(match.group(1));
            return () -> controller.getModel(modelId);
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/RouteController.java" line="110">

---

# handleProxyRequest Method

The handleProxyRequest method in the RouteController class is responsible for handling requests that are to be proxied to the origin server. This method copies headers from the original request to the proxy request, sets the content length of the proxy request, and sends the request to the origin server.

```java
    private void handleProxyRequest(HttpClientRequest proxyRequest) {
        log.info("Connected to origin: {}", proxyRequest.connection().remoteAddress());

        HttpServerRequest request = context.getRequest();
        context.setProxyRequest(proxyRequest);

        Upstream upstream = context.getUpstreamRoute().get();
        ProxyUtil.copyHeaders(request.headers(), proxyRequest.headers());
        proxyRequest.putHeader(Proxy.HEADER_API_KEY, upstream.getKey());

        Buffer proxyRequestBody = context.getRequestBody();
        proxyRequest.putHeader(HttpHeaders.CONTENT_LENGTH, Integer.toString(proxyRequestBody.length()));

        proxyRequest.send(proxyRequestBody)
                .onSuccess(this::handleProxyResponse)
                .onFailure(this::handleProxyRequestError);
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentController.java" line="62">

---

# hasAccess Method

The hasAccess method in the DeploymentController class is used to check if a client has access to a specific deployment. This method checks if the client has access based on the limits set in the client's role and the user roles associated with the deployment.

```java
    public static boolean hasAccess(ProxyContext context, Deployment deployment) {
        return hasAssessByLimits(context, deployment) && hasAccessByUserRoles(context, deployment);
    }
```

---

</SwmSnippet>

# Proxy Application Endpoints

Understanding Proxy Application Endpoints

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ApplicationController.java" line="26">

---

## getApplication Endpoint

The `getApplication` method in the `ApplicationController` class handles the GET request to retrieve a specific application based on its ID. It checks if the application exists and if the user has access to it. If both conditions are met, it responds with the application data; otherwise, it responds with an appropriate HTTP status code.

```java
    private final boolean includeCustomApplications;

    public ApplicationController(ProxyContext context, Proxy proxy) {
        this.context = context;
        this.vertx = proxy.getVertx();
        this.customApplicationService = proxy.getCustomApplicationService();
        this.includeCustomApplications = customApplicationService.includeCustomApplications();
    }

    public Future<?> getApplication(String applicationId) {
        Config config = context.getConfig();
        Application application = config.getApplications().get(applicationId);

        Future<Application> applicationFuture;
        if (application != null) {
            if (!DeploymentController.hasAccess(context, application)) {
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentController.java" line="44">

---

## getDeployments Endpoint

The `getDeployments` method in the `DeploymentController` class handles the GET request to retrieve all deployments that the user has access to. It iterates over all the deployments and checks if the user has access to each one. If the user has access, it adds the deployment data to the response.

```java
    public Future<?> getDeployments() {
        Config config = context.getConfig();
        List<DeploymentData> deployments = new ArrayList<>();

        for (Model model : config.getModels().values()) {
            if (hasAccess(context, model)) {
                DeploymentData deployment = createDeployment(model);
                deployments.add(deployment);
            }
        }

        ListData<DeploymentData> list = new ListData<>();
        list.setData(deployments);

        context.respond(HttpStatus.OK, list);
        return Future.succeededFuture();
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
