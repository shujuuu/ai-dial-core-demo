---
title: Functional Operations Overview
---
<SwmSnippet path="/src/main/java/com/epam/aidial/core/function/BaseFunction.java" line="6">

---

The `BaseFunction` class is an abstract class that implements the `Function` interface from `java.util.function`. This class serves as the base for all functional operations in the application.

```java
import java.util.function.Function;

public abstract class BaseFunction<T> implements Function<T, Throwable> {
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/function/CollectAttachmentsFn.java" line="23">

---

`CollectAttachmentsFn` is a concrete class that extends `BaseFunction`. It is responsible for collecting attached files from the chat completion request and storing the result to API key data. This function is an example of how functional operations are implemented in the application.

```java
@Slf4j
public class CollectAttachmentsFn extends BaseFunction<ObjectNode> {
    public CollectAttachmentsFn(Proxy proxy, ProxyContext context) {
        super(proxy, context);
    }

    @Override
    public Throwable apply(ObjectNode tree) {
        try {
            ProxyUtil.collectAttachedFiles(tree, this::processAttachedFile);
            // assign api key data after processing attachments
            ApiKeyData destApiKeyData = context.getProxyApiKeyData();
            proxy.getApiKeyStore().assignPerRequestApiKey(destApiKeyData);
            return null;
        } catch (HttpException e) {
            context.respond(e.getStatus(), e.getMessage());
            log.warn("Can't collect attached files. Trace: {}. Span: {}. Error: {}",
                    context.getTraceId(), context.getSpanId(), e.getMessage());
            return e;
        } catch (Throwable e) {
            context.respond(HttpStatus.BAD_REQUEST);
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentPostController.java" line="70">

---

Functional operations like `CollectAttachmentsFn` are used in various parts of the application. In this instance, `CollectAttachmentsFn` is used as part of the enhancement functions in the `DeploymentPostController`.

```java
        this.enhancementFunctions = List.of(new CollectAttachmentsFn(proxy, context),
                new ApplyDefaultDeploymentSettingsFn(proxy, context),
```

---

</SwmSnippet>

# Unified API

The unified API is a key functionality of this project. It allows for interaction with different chat completion and embedding models, assistants, and applications. This makes it easier to manage and maintain these different components, as they can all be accessed and controlled through a single API.

# Deployment on Kubernetes

Another important functionality is the ability to be deployed on a Kubernetes cluster. This allows for easy scaling and management of the project, making it suitable for large-scale applications.

# Support for Various Storage Options

The project also supports various storage options including AWS S3, Google Cloud Storage, and Azure Blob Store. This allows for a wide range of data storage and retrieval options, making it versatile and adaptable to different use cases.

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
