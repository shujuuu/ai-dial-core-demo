---
title: Enhancement and Response Flow
---
This document will cover the process of enhancing assistant requests and responding to them in the ai-dial-core-demo project. We'll cover:

1. The application of the enhancement function
2. The response process
3. The end of the response process.

```mermaid
graph TD;
subgraph src/main/java/com/epam/aidial/core
  apply:::mainFlowStyle --> respond
end
subgraph src/main/java/com/epam/aidial/core
  apply:::mainFlowStyle --> enhanceAssistantRequest
end
subgraph src/main/java/com/epam/aidial/core
  respond:::mainFlowStyle --> end
end

classDef mainFlowStyle color:#000000,fill:#7CB9F4
classDef rootsStyle color:#000000,fill:#00FFF4
classDef Style1 color:#000000,fill:#00FFAA
classDef Style2 color:#000000,fill:#FFFF00
classDef Style3 color:#000000,fill:#AA7CB9
```

<SwmSnippet path="/src/main/java/com/epam/aidial/core/function/enhancement/EnhanceAssistantRequestFn.java" line="1">

---

# The application of the enhancement function

The `apply` function in `EnhanceAssistantRequestFn.java` is the starting point of this flow. It is responsible for applying enhancements to the assistant request.

```java
package com.epam.aidial.core.function.enhancement;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/Proxy.java" line="214">

---

# The response process

After the request has been enhanced, the `respond` function in `Proxy.java` is called. This function is responsible for setting the HTTP status code and initiating the end of the response.

```java
    private void respond(HttpServerRequest request, HttpStatus status) {
        request.response().setStatusCode(status.getCode()).end();
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/storage/BlobWriteStream.java" line="94">

---

# The end of the response process

The `end` function in `BlobWriteStream.java` is the final step in this flow. It handles the completion of the response, including error handling and success confirmation.

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
