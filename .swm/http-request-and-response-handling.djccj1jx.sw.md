---
title: HTTP Request and Response Handling
---
This document will cover the process of handling HTTP requests and responses in the ai-dial-core-demo project. We'll cover:

1. How the `handle` function processes incoming requests
2. How the `handleRequest` function validates the request and initiates authorization
3. How the `respond` function sends HTTP responses
4. How the `select` function selects the appropriate controller based on the HTTP method
5. How the `handle` function in the `UploadFileController` handles file uploads
6. How the `getApplications` function retrieves application data

```mermaid
graph TD;
subgraph src/main/java/com/epam/aidial/core
  end:::mainFlowStyle --> handle
end
subgraph src/main/java/com/epam/aidial/core
  handle:::mainFlowStyle --> handleRequest
end
subgraph src/main/java/com/epam/aidial/core
  handleRequest:::mainFlowStyle --> respond
end
subgraph src/main/java/com/epam/aidial/core
  handleRequest:::mainFlowStyle --> processAuthorizationResult
end
subgraph src/main/java/com/epam/aidial/core/controller
  processAuthorizationResult:::mainFlowStyle --> select
end
subgraph src/main/java/com/epam/aidial/core/controller
  select:::mainFlowStyle --> selectDelete
end
subgraph src/main/java/com/epam/aidial/core/controller
  select:::mainFlowStyle --> selectPost
end
subgraph src/main/java/com/epam/aidial/core/controller
  select:::mainFlowStyle --> selectPut
end
subgraph src/main/java/com/epam/aidial/core/controller
  select:::mainFlowStyle --> selectGet
end
subgraph src/main/java/com/epam/aidial/core/controller
  selectDelete --> handle
end
subgraph src/main/java/com/epam/aidial/core/controller
  selectPost --> handle
end
subgraph src/main/java/com/epam/aidial/core/controller
  selectPut --> handle
end
subgraph src/main/java/com/epam/aidial/core/controller
  selectGet:::mainFlowStyle --> handle
end
subgraph src/main/java/com/epam/aidial/core/controller
  selectGet:::mainFlowStyle --> getApplication
end
subgraph src/main/java/com/epam/aidial/core/controller
  handle --> getMetadata
end
subgraph src/main/java/com/epam/aidial/core/controller
  getApplication:::mainFlowStyle --> getApplications
end
subgraph src/main/java/com/epam/aidial/core/service
  getApplications:::mainFlowStyle --> getSharedApplications
end
subgraph src/main/java/com/epam/aidial/core/service
  getApplications:::mainFlowStyle --> getOwnCustomApplications
end
subgraph src/main/java/com/epam/aidial/core/service
  getApplications:::mainFlowStyle --> getPublicApplications
end
subgraph src/main/java/com/epam/aidial/core/service
  getSharedApplications --> fetchApplicationsRecursively
end
subgraph src/main/java/com/epam/aidial/core/service
  getOwnCustomApplications --> fetchApplicationsRecursively
end
subgraph src/main/java/com/epam/aidial/core/service
  getPublicApplications --> filterForbidden
end

classDef mainFlowStyle color:#000000,fill:#7CB9F4
classDef rootsStyle color:#000000,fill:#00FFF4
classDef Style1 color:#000000,fill:#00FFAA
classDef Style2 color:#000000,fill:#FFFF00
classDef Style3 color:#000000,fill:#AA7CB9
```

<SwmSnippet path="/src/main/java/com/epam/aidial/core/Proxy.java" line="94">

---

# Handling Incoming Requests

The `handle` function is the entry point for processing incoming HTTP requests. It calls the `handleRequest` function to process the request and handles any errors that may occur.

```java
    @Override
    public void handle(HttpServerRequest request) {
        try {
            handleRequest(request);
        } catch (Throwable e) {
            handleError(e, request);
        }
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/Proxy.java" line="108">

---

# Validating the Request and Initiating Authorization

The `handleRequest` function validates the HTTP request, checks for the presence of API keys or authorization headers, and initiates the authorization process. If the request is valid and authorized, it calls the `processAuthorizationResult` function.

```java
    /**
     * Called when proxy received the request headers from a client.
     */
    private void handleRequest(HttpServerRequest request) {
        if (request.version() != HttpVersion.HTTP_1_1) {
            respond(request, HttpStatus.HTTP_VERSION_NOT_SUPPORTED);
            return;
        }

        HttpMethod requestMethod = request.method();
        if (!ALLOWED_HTTP_METHODS.contains(requestMethod)) {
            respond(request, HttpStatus.METHOD_NOT_ALLOWED);
            return;
        }

        String contentType = request.getHeader(HttpHeaders.CONTENT_TYPE);
        int contentLength = ProxyUtil.contentLength(request, 1024);
        if (contentType != null && contentType.startsWith("multipart/form-data")) {
            if (contentLength > FILES_REQUEST_BODY_MAX_SIZE_BYTES) {
                respond(request, HttpStatus.REQUEST_ENTITY_TOO_LARGE, "Request body is too large");
                return;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/Proxy.java" line="214">

---

# Sending HTTP Responses

The `respond` function is used to send HTTP responses with the specified status code.

```java
    private void respond(HttpServerRequest request, HttpStatus status) {
        request.response().setStatusCode(status.getCode()).end();
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="63">

---

# Selecting the Appropriate Controller

The `select` function selects the appropriate controller based on the HTTP method of the request. It calls the corresponding `selectGet`, `selectPost`, `selectDelete`, or `selectPut` function.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/UploadFileController.java" line="22">

---

# Handling File Uploads

The `handle` function in the `UploadFileController` handles file uploads. It validates the resource path, sets up a `BlobWriteStream` for the file, and sends a response with the file metadata upon successful upload.

```java
    @Override
    protected Future<?> handle(ResourceDescription resource, boolean hasWriteAccess) {
        if (resource.isFolder()) {
            return context.respond(HttpStatus.BAD_REQUEST, "File name is missing");
        }

        if (!ResourceDescription.isValidResourcePath(resource)) {
            return context.respond(HttpStatus.BAD_REQUEST, "Resource name and/or parent folders must not end with .(dot)");
        }

        Promise<Void> result = Promise.promise();
        context.getRequest()
                .setExpectMultipart(true)
                .uploadHandler(upload -> {
                    String contentType = upload.contentType();
                    Pipe<Buffer> pipe = new PipeImpl<>(upload).endOnFailure(false);
                    BlobWriteStream writeStream = new BlobWriteStream(
                            proxy.getVertx(),
                            proxy.getStorage(),
                            resource,
                            contentType);
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ApplicationController.java" line="62">

---

# Retrieving Application Data

The `getApplications` function retrieves application data. It fetches data for both standard and custom applications, and sends a response with the application data.

```java
    public Future<?> getApplications() {
        Config config = context.getConfig();
        List<ApplicationData> applications = new ArrayList<>();

        for (Application application : config.getApplications().values()) {
            if (DeploymentController.hasAccess(context, application)) {
                ApplicationData data = ApplicationUtil.mapApplication(application);
                applications.add(data);
            }
        }

        ListData<ApplicationData> list = new ListData<>();
        list.setData(applications);

        if (includeCustomApplications) {
            vertx.executeBlocking(() -> {
                List<Application> ownCustomApplications = customApplicationService.getOwnCustomApplications(context);
                for (Application application : ownCustomApplications) {
                    ApplicationData data = ApplicationUtil.mapApplication(application);
                    applications.add(data);
                }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="flows"><sup>Powered by [Swimm](/)</sup></SwmMeta>