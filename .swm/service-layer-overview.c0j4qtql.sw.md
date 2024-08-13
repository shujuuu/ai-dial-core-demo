---
title: Service Layer Overview
---
The Service Layer in ai-dial-core-demo is a crucial architectural component that encapsulates the application's business logic. It is composed of various services, each responsible for a specific functionality.

For instance, the `CustomApplicationService` is a part of the Service Layer that manages the application's custom operations. It uses other services like `EncryptionService`, `ResourceService`, `ShareService`, and `AccessService` to perform its tasks.

Another key service is the `ResourceOperationService`, which handles operations related to resources such as moving and deleting resources. It collaborates with `ResourceService`, `BlobStorage`, `InvitationService`, and `ShareService` to accomplish its functions.

The `ResourceService` is another important service in the Service Layer. It is responsible for managing resources, including their creation, deletion, and retrieval. It uses a `BlobStorage` for storing the resources and a `LockService` for managing access to resources.

Overall, the Service Layer in ai-dial-core-demo is designed to ensure a separation of concerns, where each service is responsible for a specific set of functionalities. This design makes the codebase easier to maintain and extend.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/CustomApplicationService.java" line="1">

---

# CustomApplicationService

The `CustomApplicationService` is a part of the Service Layer that manages the application's custom operations. It uses other services like `EncryptionService`, `ResourceService`, `ShareService`, and `AccessService` to perform its tasks.

```java
package com.epam.aidial.core.service;

import com.epam.aidial.core.ProxyContext;
import com.epam.aidial.core.config.Application;
import com.epam.aidial.core.data.ListSharedResourcesRequest;
import com.epam.aidial.core.data.MetadataBase;
import com.epam.aidial.core.data.NodeType;
import com.epam.aidial.core.data.ResourceFolderMetadata;
import com.epam.aidial.core.data.ResourceType;
import com.epam.aidial.core.data.SharedResourcesResponse;
import com.epam.aidial.core.security.AccessService;
import com.epam.aidial.core.security.EncryptionService;
import com.epam.aidial.core.storage.BlobStorageUtil;
import com.epam.aidial.core.storage.ResourceDescription;
import com.epam.aidial.core.util.ProxyUtil;
import io.vertx.core.json.JsonObject;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import java.util.ArrayList;
import java.util.List;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ResourceOperationService.java" line="1">

---

# ResourceOperationService

The `ResourceOperationService` handles operations related to resources such as moving and deleting resources. It collaborates with `ResourceService`, `BlobStorage`, `InvitationService`, and `ShareService` to accomplish its functions.

```java
package com.epam.aidial.core.service;

import com.epam.aidial.core.data.ResourceType;
import com.epam.aidial.core.storage.BlobStorage;
import com.epam.aidial.core.storage.ResourceDescription;
import com.epam.aidial.core.util.ResourceUtil;
import lombok.AllArgsConstructor;

@AllArgsConstructor
public class ResourceOperationService {

    private final ResourceService resourceService;
    private final BlobStorage storage;
    private final InvitationService invitationService;
    private final ShareService shareService;

    public void moveResource(String bucket, String location, ResourceDescription source, ResourceDescription destination, boolean overwriteIfExists) {
        if (source.isFolder() || destination.isFolder()) {
            throw new IllegalArgumentException("Moving folders is not supported");
        }

```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ResourceService.java" line="1">

---

# ResourceService

The `ResourceService` is responsible for managing resources, including their creation, deletion, and retrieval. It uses a `BlobStorage` for storing the resources and a `LockService` for managing access to resources.

```java
package com.epam.aidial.core.service;

import com.epam.aidial.core.data.MetadataBase;
import com.epam.aidial.core.data.ResourceFolderMetadata;
import com.epam.aidial.core.data.ResourceItemMetadata;
import com.epam.aidial.core.storage.BlobStorage;
import com.epam.aidial.core.storage.BlobStorageUtil;
import com.epam.aidial.core.storage.ResourceDescription;
import com.epam.aidial.core.util.Compression;
import io.vertx.core.Vertx;
import io.vertx.core.json.JsonObject;
import lombok.Getter;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.tuple.Pair;
import org.jclouds.blobstore.domain.Blob;
import org.jclouds.blobstore.domain.BlobMetadata;
import org.jclouds.blobstore.domain.PageSet;
import org.jclouds.blobstore.domain.StorageMetadata;
import org.jclouds.blobstore.domain.StorageType;
import org.redisson.api.RMap;
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
