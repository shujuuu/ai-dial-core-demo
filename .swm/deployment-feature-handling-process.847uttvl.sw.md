---
title: Deployment Feature Handling Process
---
This document will cover the process of handling deployment features in the ai-dial-core-demo project. We'll cover:

1. The initiation of the process in the `handle` function.
2. The collection and processing of attachments.
3. The assignment of per-request API keys.
4. The response handling in case of exceptions.

```mermaid
graph TD;
subgraph src/main/java/com/epam/aidial/core
  handle:::mainFlowStyle --> apply
end
subgraph src/main/java/com/epam/aidial/core
  handle:::mainFlowStyle --> selectDeployment
end
subgraph src/main/java/com/epam/aidial/core
  apply:::mainFlowStyle --> respond
end

classDef mainFlowStyle color:#000000,fill:#7CB9F4
classDef rootsStyle color:#000000,fill:#00FFF4
classDef Style1 color:#000000,fill:#00FFAA
classDef Style2 color:#000000,fill:#FFFF00
classDef Style3 color:#000000,fill:#AA7CB9
```

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentFeatureController.java" line="1">

---

# The initiation of the process in the `handle` function

The `handle` function in `DeploymentFeatureController.java` is the entry point for this flow. It calls the `apply` function in `CollectAttachmentsFn.java` and `selectDeployment` function in `Config.java`.

```java
package com.epam.aidial.core.controller;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/function/CollectAttachmentsFn.java" line="29">

---

# The collection and processing of attachments

The `apply` function in `CollectAttachmentsFn.java` is responsible for collecting and processing attached files. It uses the `ProxyUtil.collectAttachedFiles` method to collect the files and process them with `this::processAttachedFile`. In case of exceptions, it logs a warning and responds with the appropriate HTTP status.

```java
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
            log.warn("Can't collect attached files. Trace: {}. Span: {}. Error: {}",
                    context.getTraceId(), context.getSpanId(), e.getMessage());
            return e;
        }
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/function/CollectAttachmentsFn.java" line="34">

---

# The assignment of per-request API keys

After processing the attachments, the `apply` function assigns per-request API keys using `proxy.getApiKeyStore().assignPerRequestApiKey(destApiKeyData)`.

```java
            ApiKeyData destApiKeyData = context.getProxyApiKeyData();
            proxy.getApiKeyStore().assignPerRequestApiKey(destApiKeyData);
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/Proxy.java" line="214">

---

# The response handling in case of exceptions

The `respond` function in `Proxy.java` is used to send a response with the appropriate HTTP status in case of exceptions during the process.

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
