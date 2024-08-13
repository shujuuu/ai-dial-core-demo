---
title: HTTP Request Handling Process
---
This document will cover the process of handling HTTP requests in the ai-dial-core-demo project. The process includes the following steps:

1. Applying the enhancement function to the assistant request
2. Responding to the HTTP request
3. Ending the blob write stream.

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

# Applying the enhancement function to the assistant request

The `apply` function in `EnhanceAssistantRequestFn.java` is the starting point of the flow. It is responsible for applying the enhancement function to the assistant request.

```java
package com.epam.aidial.core.function.enhancement;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/Proxy.java" line="214">

---

# Responding to the HTTP request

The `respond` function in `Proxy.java` is called next. It sets the HTTP status code and ends the response.

```java
    private void respond(HttpServerRequest request, HttpStatus status) {
        request.response().setStatusCode(status.getCode()).end();
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/storage/BlobWriteStream.java" line="94">

---

# Ending the blob write stream

The `end` function in `BlobWriteStream.java` is the final step in the flow. It handles the completion of the blob write stream, including error handling and multipart upload if necessary.

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
