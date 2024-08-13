---
title: Exploring Application Selection in Controller Selector
---
Proxy Application Selection refers to the process of choosing the appropriate application to handle a specific request in the Controller Selection. This is achieved through the `select` method in the `ControllerSelector` class, which uses HTTP methods (GET, POST, DELETE, PUT) to determine the appropriate controller for handling the request. The selection process involves matching the request path against a set of predefined patterns, each corresponding to a specific application. Once a match is found, the corresponding controller is instantiated and returned to handle the request.

For instance, in the `selectGet` method, the request path is matched against patterns for deployments, models, addons, assistants, and applications. If a match is found, the corresponding controller is instantiated and returned. The instantiated controller is then used to handle the request, which could involve fetching data from a deployment, model, addon, assistant, or application.

Similarly, the `selectPost`, `selectDelete`, and `selectPut` methods handle POST, DELETE, and PUT requests respectively. They also match the request path against a set of patterns and return the corresponding controller to handle the request.

In summary, Proxy Application Selection is a crucial part of the Controller Selection process, ensuring that each request is handled by the appropriate application.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="63">

---

# Controller Selector

The `select` method in the `ControllerSelector` class is the entry point for Proxy Application Selection. It uses the HTTP method of the request to determine which method (`selectGet`, `selectPost`, `selectDelete`, or `selectPut`) to call for further processing.

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

# Selecting GET Requests

The `selectGet` method handles GET requests. It matches the request path against a set of patterns corresponding to different applications. If a match is found, it instantiates the corresponding controller and returns it to handle the request.

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

# Selecting POST Requests

The `selectPost` method handles POST requests in a similar manner to `selectGet`. It matches the request path against a set of patterns and returns the corresponding controller to handle the request.

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

# Selecting DELETE Requests

The `selectDelete` method handles DELETE requests. It matches the request path against a set of patterns and returns the corresponding controller to handle the request.

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

# Selecting PUT Requests

The `selectPut` method handles PUT requests. It matches the request path against a set of patterns and returns the corresponding controller to handle the request.

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

# Proxy Application Selection

Understanding Proxy Application Selection

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/UploadFileController.java" line="1">

---

## UploadFileController

The `UploadFileController` class handles the uploading of files. It extends the `AccessControlBaseController` which provides basic access control functionalities. The `handle` method is overridden to provide specific functionality for file upload. It checks if the resource is a folder, validates the resource path, and then initiates the file upload process.

```java
package com.epam.aidial.core.controller;

import com.epam.aidial.core.Proxy;
import com.epam.aidial.core.ProxyContext;
import com.epam.aidial.core.storage.BlobWriteStream;
import com.epam.aidial.core.storage.ResourceDescription;
import com.epam.aidial.core.util.HttpStatus;
import io.vertx.core.Future;
import io.vertx.core.Promise;
import io.vertx.core.buffer.Buffer;
import io.vertx.core.streams.Pipe;
import io.vertx.core.streams.impl.PipeImpl;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class UploadFileController extends AccessControlBaseController {

    public UploadFileController(Proxy proxy, ProxyContext context) {
        super(proxy, context, true);
    }

```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeleteFileController.java" line="1">

---

## DeleteFileController

The `DeleteFileController` class handles the deletion of files. Similar to `UploadFileController`, it extends the `AccessControlBaseController`. The `handle` method is overridden to provide specific functionality for file deletion. It checks if the resource is a folder, gets the absolute file path, and then initiates the file deletion process.

```java
package com.epam.aidial.core.controller;

import com.epam.aidial.core.Proxy;
import com.epam.aidial.core.ProxyContext;
import com.epam.aidial.core.service.InvitationService;
import com.epam.aidial.core.service.LockService;
import com.epam.aidial.core.service.ShareService;
import com.epam.aidial.core.storage.BlobStorage;
import com.epam.aidial.core.storage.ResourceDescription;
import com.epam.aidial.core.util.HttpStatus;
import io.vertx.core.Future;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class DeleteFileController extends AccessControlBaseController {

    private final ShareService shareService;
    private final InvitationService invitationService;
    private final LockService lockService;

    public DeleteFileController(Proxy proxy, ProxyContext context) {
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
