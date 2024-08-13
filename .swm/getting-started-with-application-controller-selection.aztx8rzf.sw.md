---
title: Getting Started with Application Controller Selection
---
Proxy Application Selection refers to the process of choosing the appropriate application controller based on the incoming request. This is done in the `ControllerSelector` class, specifically in the `select` method. The `select` method uses the HTTP method and request path to determine which controller to use. For example, if the HTTP method is GET, the `selectGet` method is called, which checks the request path against a series of predefined patterns. If a match is found, the corresponding controller is selected and returned. If no match is found, a default `RouteController` is returned. The selected controller is then used to handle the request in the `Proxy` class.

The `ApplicationController` class is one of the possible controllers that can be selected. It is used to handle requests related to applications. The `getApplication` method in this class is used to retrieve a specific application based on its ID. If the application is not found or the user does not have access to it, an appropriate HTTP response is returned.

The `ControllerSelector` class also contains methods for handling POST, DELETE, and PUT requests, namely `selectPost`, `selectDelete`, and `selectPut`. These methods work similarly to `selectGet`, matching the request path against predefined patterns to select the appropriate controller.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="63">

---

# Controller Selection

This is the `select` method in the `ControllerSelector` class. It checks the HTTP method of the request and calls the corresponding method (`selectGet`, `selectPost`, `selectDelete`, or `selectPut`).

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ApplicationController.java" line="25">

---

# ApplicationController

The `ApplicationController` class is one of the possible controllers that can be selected. It is used to handle requests related to applications. The `getApplication` method in this class is used to retrieve a specific application based on its ID.

```java
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
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="81">

---

# Handling GET Requests

This is the `selectGet` method in the `ControllerSelector` class. It checks the request path against a series of predefined patterns. If a match is found, the corresponding controller is selected and returned.

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

# Handling POST Requests

This is the `selectPost` method in the `ControllerSelector` class. It works similarly to `selectGet`, matching the request path against predefined patterns to select the appropriate controller.

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

# Handling DELETE Requests

This is the `selectDelete` method in the `ControllerSelector` class. It works similarly to `selectGet` and `selectPost`, matching the request path against predefined patterns to select the appropriate controller.

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

# Proxy Application Selection Endpoints

Understanding Proxy Application Selection

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentController.java" line="27">

---

## getDeployment Endpoint

The `getDeployment` method in the `DeploymentController` class is used to handle requests for a specific deployment. It first checks if the deployment exists and if the user has access to it. If the deployment is not found or the user does not have access, an appropriate HTTP response is returned. If the deployment is found and the user has access, a `DeploymentData` object is created and returned in the response.

```java
    public Future<?> getDeployment(String deploymentId) {
        Config config = context.getConfig();
        Model model = config.getModels().get(deploymentId);

        if (model == null) {
            return context.respond(HttpStatus.NOT_FOUND);
        }

        if (!DeploymentController.hasAccess(context, model)) {
            return context.respond(HttpStatus.FORBIDDEN);
        }

        DeploymentData data = createDeployment(model);
        context.respond(HttpStatus.OK, data);
        return Future.succeededFuture();
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ModelController.java" line="43">

---

## getModels Endpoint

The `getModels` method in the `ModelController` class is used to handle requests for all models. It iterates over all the models in the configuration and checks if the user has access to each model. If the user has access, a `ModelData` object is created and added to the response list. The list of `ModelData` objects is then returned in the response.

```java
        Config config = context.getConfig();
        List<ModelData> models = new ArrayList<>();

        for (Model model : config.getModels().values()) {
            if (DeploymentController.hasAccess(context, model)) {
                ModelData data = createModel(model);
                models.add(data);
            }
        }

        ListData<ModelData> list = new ListData<>();
        list.setData(models);

        context.respond(HttpStatus.OK, list);
        return Future.succeededFuture();
    }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
