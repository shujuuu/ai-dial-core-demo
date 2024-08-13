---
title: Application Proxy Selection Overview
---
Proxy Application Selection refers to the process of choosing the appropriate controller to handle a specific request in the AI Dial Core. This is achieved through the `select` method in the `ControllerSelector` class, which uses the HTTP method and request path to determine the appropriate controller.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="63">

---

The `select` method in the `ControllerSelector` class is responsible for selecting the appropriate controller based on the HTTP method and request path. It uses the `selectGet`, `selectPost`, `selectDelete`, and `selectPut` methods to handle different HTTP methods.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="81">

---

The `selectGet` method is used to handle GET requests. It uses regular expressions to match the request path to various patterns, each corresponding to a different controller. For example, if the request path matches the `PATTERN_APPLICATION` pattern, the `ApplicationController` is selected.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="214">

---

The `selectPost` method is used to handle POST requests. Similar to `selectGet`, it uses regular expressions to match the request path to various patterns, each corresponding to a different controller.

```java
    private static Controller selectPost(Proxy proxy, ProxyContext context, String path) {
        Matcher match = match(PATTERN_POST_DEPLOYMENT, path);
        if (match != null) {
            String deploymentId = UrlUtil.decodePath(match.group(1));
            String deploymentApi = UrlUtil.decodePath(match.group(2));
            DeploymentPostController controller = new DeploymentPostController(proxy, context);
            return () -> controller.handle(deploymentId, deploymentApi);
        }

        match = match(PATTERN_RATE_RESPONSE, path);
        if (match != null) {
            String deploymentId = UrlUtil.decodePath(match.group(1));

            Function<Deployment, String> getter = (model) -> Optional.ofNullable(model)
                    .map(Deployment::getFeatures)
                    .map(Features::getRateEndpoint)
                    .orElse(null);

            DeploymentFeatureController controller = new DeploymentFeatureController(proxy, context);
            return () -> controller.handle(deploymentId, getter, false);
        }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="320">

---

The `selectDelete` method is used to handle DELETE requests. It uses regular expressions to match the request path to various patterns, each corresponding to a different controller.

```java
    private static Controller selectDelete(Proxy proxy, ProxyContext context, String path) {
        Matcher match = match(PATTERN_FILES, path);
        if (match != null) {
            DeleteFileController controller = new DeleteFileController(proxy, context);
            return () -> controller.handle(resourcePath(path));
        }

        match = match(PATTERN_RESOURCE, path);
        if (match != null) {
            ResourceController controller = new ResourceController(proxy, context, false);
            return () -> controller.handle(resourcePath(path));
        }

        match = match(INVITATION, path);
        if (match != null) {
            String invitationId = UrlUtil.decodePath(match.group(1));
            InvitationController controller = new InvitationController(proxy, context);
            return () -> controller.deleteInvitation(invitationId);
        }

        return null;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="343">

---

The `selectPut` method is used to handle PUT requests. It uses regular expressions to match the request path to various patterns, each corresponding to a different controller.

```java
    private static Controller selectPut(Proxy proxy, ProxyContext context, String path) {
        Matcher match = match(PATTERN_FILES, path);
        if (match != null) {
            UploadFileController controller = new UploadFileController(proxy, context);
            return () -> controller.handle(resourcePath(path));
        }

        match = match(PATTERN_RESOURCE, path);
        if (match != null) {
            ResourceController controller = new ResourceController(proxy, context, false);
            return () -> controller.handle(resourcePath(path));
        }

        return null;
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="63">

---

# Select Method

The `select` method is the core of the Proxy Application Selection mechanism. It checks the HTTP method of the request and calls the appropriate method (`selectGet`, `selectPost`, `selectDelete`, or `selectPut`) based on it. If no controller is selected by these methods, it defaults to a `RouteController`.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="81">

---

# SelectGet Method

The `selectGet` method is called when the HTTP method of the request is GET. It uses a series of pattern matching operations to determine which controller should handle the request. Each pattern corresponds to a different type of request, and the method returns a controller that is capable of handling that type of request.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="214">

---

# SelectPost Method

The `selectPost` method is similar to `selectGet`, but it is used for POST requests. It also uses pattern matching to determine the appropriate controller for the request.

```java
    private static Controller selectPost(Proxy proxy, ProxyContext context, String path) {
        Matcher match = match(PATTERN_POST_DEPLOYMENT, path);
        if (match != null) {
            String deploymentId = UrlUtil.decodePath(match.group(1));
            String deploymentApi = UrlUtil.decodePath(match.group(2));
            DeploymentPostController controller = new DeploymentPostController(proxy, context);
            return () -> controller.handle(deploymentId, deploymentApi);
        }

        match = match(PATTERN_RATE_RESPONSE, path);
        if (match != null) {
            String deploymentId = UrlUtil.decodePath(match.group(1));

            Function<Deployment, String> getter = (model) -> Optional.ofNullable(model)
                    .map(Deployment::getFeatures)
                    .map(Features::getRateEndpoint)
                    .orElse(null);

            DeploymentFeatureController controller = new DeploymentFeatureController(proxy, context);
            return () -> controller.handle(deploymentId, getter, false);
        }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="320">

---

# SelectDelete Method

The `selectDelete` method is used for DELETE requests. Like the other `select` methods, it uses pattern matching to determine the appropriate controller.

```java
    private static Controller selectDelete(Proxy proxy, ProxyContext context, String path) {
        Matcher match = match(PATTERN_FILES, path);
        if (match != null) {
            DeleteFileController controller = new DeleteFileController(proxy, context);
            return () -> controller.handle(resourcePath(path));
        }

        match = match(PATTERN_RESOURCE, path);
        if (match != null) {
            ResourceController controller = new ResourceController(proxy, context, false);
            return () -> controller.handle(resourcePath(path));
        }

        match = match(INVITATION, path);
        if (match != null) {
            String invitationId = UrlUtil.decodePath(match.group(1));
            InvitationController controller = new InvitationController(proxy, context);
            return () -> controller.deleteInvitation(invitationId);
        }

        return null;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ApplicationController.java" line="21">

---

# ApplicationController Class

The `ApplicationController` class is an example of a controller that can be selected by the Proxy Application Selection mechanism. It has methods for handling different types of requests related to applications.

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

    public Future<?> getApplication(String applicationId) {
```

---

</SwmSnippet>

# Model and Application Controllers

Understanding Model and Application Controllers

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ModelController.java" line="20">

---

## ModelController

The ModelController class handles operations related to models. It includes methods to get a specific model by its ID (`getModel`) and to get a list of all models (`getModels`). The `getModel` method retrieves a model from the configuration and checks if the client has access to it. If the model is not found or the client doesn't have access, it responds with an appropriate HTTP status code. The `getModels` method retrieves all models from the configuration that the client has access to and responds with a list of these models.

```java
@RequiredArgsConstructor
public class ModelController {

    private final ProxyContext context;

    public Future<?> getModel(String modelId) {
        Config config = context.getConfig();
        Model model = config.getModels().get(modelId);

        if (model == null) {
            return context.respond(HttpStatus.NOT_FOUND);
        }

        if (!DeploymentController.hasAccess(context, model)) {
            return context.respond(HttpStatus.FORBIDDEN);
        }

        ModelData data = createModel(model);
        context.respond(HttpStatus.OK, data);
        return Future.succeededFuture();
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ApplicationController.java" line="20">

---

## ApplicationController

The ApplicationController class handles operations related to applications. It includes methods to get a specific application by its ID (`getApplication`) and to get a list of all applications (`getApplications`). The `getApplication` method retrieves an application from the configuration or from custom applications and checks if the client has access to it. If the application is not found or the client doesn't have access, it responds with an appropriate HTTP status code. The `getApplications` method retrieves all applications from the configuration and custom applications that the client has access to and responds with a list of these applications.

```java
@Slf4j
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

    public Future<?> getApplication(String applicationId) {
        Config config = context.getConfig();
        Application application = config.getApplications().get(applicationId);

        Future<Application> applicationFuture;
        if (application != null) {
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
