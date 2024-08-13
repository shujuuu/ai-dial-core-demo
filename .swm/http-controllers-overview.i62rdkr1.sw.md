---
title: HTTP Controllers Overview
---
Controllers in the ai-dial-core-demo project are classes that handle HTTP requests and responses. They are responsible for processing incoming requests, performing the necessary operations, and sending back the appropriate responses.

The Controller interface, defined in src/main/java/com/epam/aidial/core/controller/Controller.java, is the common interface for all HTTP controllers in the project. It declares a single method, handle(), which must be implemented by any class that acts as a controller. This method is responsible for handling the HTTP request and returning a Future object.

The ControllerSelector class, defined in src/main/java/com/epam/aidial/core/controller/ControllerSelector.java, is responsible for selecting the appropriate controller based on the incoming request. It examines the HTTP method and path of the request to determine which controller should handle it.

There are several specific controller classes in the project, each responsible for handling a specific type of request. For example, the ModelController class handles requests related to models, the ApplicationController class handles requests related to applications, and the AssistantController class handles requests related to assistants.

The AccessControlBaseController class, defined in src/main/java/com/epam/aidial/core/controller/AccessControlBaseController.java, is an abstract base class for controllers that need to perform access control. It provides common functionality for checking whether the requester has the necessary permissions to perform the requested operation.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/Controller.java" line="13">

---

# Controller Interface

The Controller interface is the common interface for all HTTP controllers in the project. It declares a single method, handle(), which must be implemented by any class that acts as a controller. This method is responsible for handling the HTTP request and returning a Future object.

```java
public interface Controller extends Serializable {

    /**
     * The controller must return non-null instance of {@link Future}.
     */
    Future<?> handle() throws Exception;
}
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ControllerSelector.java" line="63">

---

# ControllerSelector Class

The ControllerSelector class is responsible for selecting the appropriate controller based on the incoming request. It examines the HTTP method and path of the request to determine which controller should handle it.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ModelController.java" line="21">

---

# Specific Controller Classes

There are several specific controller classes in the project, each responsible for handling a specific type of request. For example, the ModelController class handles requests related to models.

```java
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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ApplicationController.java" line="21">

---

The ApplicationController class handles requests related to applications.

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
        Config config = context.getConfig();
        Application application = config.getApplications().get(applicationId);

        Future<Application> applicationFuture;
        if (application != null) {
            if (!DeploymentController.hasAccess(context, application)) {
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/AssistantController.java" line="17">

---

The AssistantController class handles requests related to assistants.

```java
@RequiredArgsConstructor
public class AssistantController {

    private final ProxyContext context;

    public Future<?> getAssistant(String assistantId) {
        Config config = context.getConfig();
        Assistant assistant = config.getAssistant().getAssistants().get(assistantId);

        if (assistant == null || ASSISTANT.equals(assistant.getName())) {
            return context.respond(HttpStatus.NOT_FOUND);
        }

        if (!DeploymentController.hasAccess(context, assistant)) {
            return context.respond(HttpStatus.FORBIDDEN);
        }

        AssistantData data = createAssistant(assistant);
        context.respond(HttpStatus.OK, data);
        return Future.succeededFuture();
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/AccessControlBaseController.java" line="14">

---

# AccessControlBaseController Class

The AccessControlBaseController class is an abstract base class for controllers that need to perform access control. It provides common functionality for checking whether the requester has the necessary permissions to perform the requested operation.

```java
@AllArgsConstructor
public abstract class AccessControlBaseController {

    final Proxy proxy;
    final ProxyContext context;
    final boolean isWriteAccess;

    public Future<?> handle(String resourceUrl) {
        ResourceDescription resource;

        try {
            resource = ResourceDescription.fromAnyUrl(resourceUrl, proxy.getEncryptionService());
        } catch (IllegalArgumentException e) {
            String errorMessage = e.getMessage() != null ? e.getMessage() : ("Invalid resource url provided: " + resourceUrl);
            context.respond(HttpStatus.BAD_REQUEST, errorMessage);
            return Future.succeededFuture();
        }

        return proxy.getVertx()
                .executeBlocking(() -> {
                    AccessService service = proxy.getAccessService();
```

---

</SwmSnippet>

# Controllers and Endpoints

Understanding Controllers and their Endpoints

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/UploadFileController.java" line="22">

---

## UploadFileController.handle

The `handle` method in `UploadFileController` is responsible for handling file upload requests. It checks if the resource is a folder and if the resource path is valid. If these conditions are met, it sets up a `BlobWriteStream` to write the uploaded file to the storage. It also handles the success and failure scenarios of the upload operation.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeleteFileController.java" line="28">

---

## DeleteFileController.handle

The `handle` method in `DeleteFileController` is responsible for handling file deletion requests. It checks if the resource is a folder and if it's not, it proceeds to delete the file from the storage. It also handles the success and failure scenarios of the deletion operation.

```java
    @Override
    protected Future<?> handle(ResourceDescription resource, boolean hasWriteAccess) {
        if (resource.isFolder()) {
            return context.respond(HttpStatus.BAD_REQUEST, "Can't delete a folder");
        }

        String absoluteFilePath = resource.getAbsoluteFilePath();

        BlobStorage storage = proxy.getStorage();
        Future<Void> result = proxy.getVertx().executeBlocking(() -> {
            String bucketName = resource.getBucketName();
            String bucketLocation = resource.getBucketLocation();
            try {
                return lockService.underBucketLock(bucketLocation, () -> {
                    invitationService.cleanUpResourceLink(bucketName, bucketLocation, resource);
                    shareService.revokeSharedResource(bucketName, bucketLocation, resource);
                    storage.delete(absoluteFilePath);
                    return null;
                });
            } catch (Exception ex) {
                log.error("Failed to delete file  %s/%s".formatted(bucketName, resource.getOriginalPath()), ex);
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
