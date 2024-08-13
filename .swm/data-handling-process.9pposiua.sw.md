---
title: Data Handling Process
---
This document will cover the process of handling data in the ai-dial-core-demo project. We'll cover:

1. How the handleData function initiates the process
2. The role of the Proxy class in handling requests
3. The selection of the appropriate controller based on the HTTP method
4. The process of handling file uploads
5. The retrieval of application data
6. The process of responding to the client.

```mermaid
graph TD;
subgraph src/main/java/com/epam/aidial/core/Proxy.java
  handleData:::mainFlowStyle --> handle
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

# Initiating the process with handleData

The `handleData` function in the `InputStreamReader` class is the entry point for this process. It reads data from an input stream and passes it to the `handle` function in the `Proxy` class for further processing.

```java
package com.epam.aidial.core.storage;

import io.netty.buffer.Unpooled;
import io.vertx.core.AsyncResult;
import io.vertx.core.Future;
import io.vertx.core.Handler;
import io.vertx.core.Vertx;
import io.vertx.core.buffer.Buffer;
import io.vertx.core.buffer.impl.BufferImpl;
import io.vertx.core.streams.Pipe;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/Proxy.java" line="94">

---

# Handling requests with the Proxy class

The `handle` function in the `Proxy` class receives the request and passes it to the `handleRequest` function. If an error occurs, it is caught and handled by the `handleError` function.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="63">

---

# Selecting the appropriate controller

The `select` function in the `ControllerSelector` class selects the appropriate controller based on the HTTP method of the request. It uses the `selectGet`, `selectPost`, `selectDelete`, and `selectPut` functions to select the appropriate controller for each HTTP method.

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

The `handle` function in the `UploadFileController` class handles file uploads. It checks if the resource is a folder and if the resource path is valid. It then creates a `BlobWriteStream` to write the uploaded data to the storage.

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

The `getApplications` function in the `ApplicationController` class retrieves application data. It gets the data for both the standard and custom applications. If the `includeCustomApplications` flag is set, it also fetches data for the custom applications.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/Proxy.java" line="214">

---

# Responding to the client

Finally, the `respond` function in the `Proxy` class sends the response back to the client. It sets the HTTP status code and ends the response.

```java
    private void respond(HttpServerRequest request, HttpStatus status) {
        request.response().setStatusCode(status.getCode()).end();
    }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="flows"><sup>Powered by [Swimm](/)</sup></SwmMeta>
