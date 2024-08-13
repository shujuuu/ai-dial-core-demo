---
title: HTTP Request and Response Handling
---
This document will cover the process of handling HTTP requests and responses in the ai-dial-core-demo project. We'll cover:

1. How the handleEnd function initiates the process
2. The role of the handle function in processing the request
3. How the handleRequest function validates the request
4. The process of responding to the request
5. The role of the ControllerSelector in determining the appropriate controller
6. The process of handling file uploads
7. The process of retrieving application data
8. The process of responding to the client.

```mermaid
graph TD;
subgraph src/main/java/com/epam/aidial/core/Proxy.java
  handleEnd:::mainFlowStyle --> handle
end
subgraph src/main/java/com/epam/aidial/core/Proxy.java
  handle:::mainFlowStyle --> handleRequest
end
subgraph src/main/java/com/epam/aidial/core/Proxy.java
  handleRequest:::mainFlowStyle --> respond
end
subgraph src/main/java/com/epam/aidial/core/Proxy.java
  handleRequest:::mainFlowStyle --> processAuthorizationResult
end
subgraph src/main/java/com/epam/aidial/core/storage
  respond:::mainFlowStyle --> end
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
  getApplications:::mainFlowStyle --> getPublicApplications
end
subgraph src/main/java/com/epam/aidial/core/service
  getApplications:::mainFlowStyle --> getOwnCustomApplications
end
subgraph src/main/java/com/epam/aidial/core/service
  getApplications:::mainFlowStyle --> getSharedApplications
end
subgraph src/main/java/com/epam/aidial/core/service
  getPublicApplications --> filterForbidden
end
subgraph src/main/java/com/epam/aidial/core/service
  getSharedApplications --> fetchApplicationsRecursively
end

classDef mainFlowStyle color:#000000,fill:#7CB9F4
classDef rootsStyle color:#000000,fill:#00FFF4
classDef Style1 color:#000000,fill:#00FFAA
classDef Style2 color:#000000,fill:#FFFF00
classDef Style3 color:#000000,fill:#AA7CB9
```

<SwmSnippet path="/src/main/java/com/epam/aidial/core/storage/InputStreamReader.java" line="1">

---

# Initiating the process with handleEnd

The process begins with the `handleEnd` function, which is not shown in the provided context. This function is likely responsible for reading input from a stream and passing it to the `handle` function in the Proxy class.

```java
package com.epam.aidial.core.storage;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/Proxy.java" line="94">

---

# Processing the request with handle

The `handle` function receives the HTTP request and passes it to the `handleRequest` function. If an error occurs during this process, it is caught and handled by the `handleError` function.

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

# Validating the request with handleRequest

The `handleRequest` function validates the HTTP request. It checks the HTTP version, the request method, and the content type and size of the request body. If the request is valid, it is passed to the `processAuthorizationResult` function.

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

# Responding to the request

The `respond` function sends a response to the client with the specified HTTP status code.

```java
    private void respond(HttpServerRequest request, HttpStatus status) {
        request.response().setStatusCode(status.getCode()).end();
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="63">

---

# Determining the appropriate controller

The `select` function in the ControllerSelector class determines the appropriate controller based on the HTTP method of the request. It calls one of the `selectGet`, `selectPost`, `selectDelete`, or `selectPut` functions, which return the appropriate controller.

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

# Handling file uploads

The `handle` function in the UploadFileController class handles file uploads. It validates the resource, sets up a write stream to the storage, and responds to the client with the metadata of the uploaded file.

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

# Retrieving application data

The `getApplications` function retrieves application data. It gets the data for both standard and custom applications, and responds to the client with this data.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/storage/BlobWriteStream.java" line="94">

---

# Responding to the client

The `end` function in the BlobWriteStream class finalizes the response to the client. It commits the multipart upload to the storage and responds to the client with the status of the upload.

```java
    @Override
    public void end(Handler<AsyncResult<Void>> handler) {
        Future<Void> result = vertx.executeBlocking(() -> {
            synchronized (BlobWriteStream.this) {
                if (exception != null) {
                    throw new RuntimeException(exception);
                }

                Buffer lastChunk = chunkBuffer.slice(0, position);
                metadata = new FileMetadata(resource, bytesHandled, contentType);
                if (mpu == null) {
                    log.info("Resource is too small for multipart upload, sending as a regular blob");
                    storage.store(resource.getAbsoluteFilePath(), contentType, lastChunk);
                } else {
                    if (position != 0) {
                        MultipartPart part = storage.storeMultipartPart(mpu, ++chunkNumber, lastChunk);
                        parts.add(part);
                    }

                    storage.completeMultipartUpload(mpu, parts);
                    log.info("Multipart upload committed, bytes handled {}", bytesHandled);
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="flows"><sup>Powered by [Swimm](/)</sup></SwmMeta>
