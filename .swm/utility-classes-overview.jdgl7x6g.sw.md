---
title: Utility Classes Overview
---
Utilities in the ai-dial-core-demo project refer to a set of helper classes that provide common functionality used across the codebase. These utilities are designed to perform specific tasks that are often required in different parts of the application, thereby promoting code reuse and reducing redundancy.

One such utility is the `ResourceUtil` class. This class provides methods to handle operations related to resources, such as checking if a resource exists, determining the type of a resource, and converting shared resources to a map. These methods are used in various parts of the application, such as the `ShareService`, `ShareController`, `ResourceOperationService`, and `SharedResources` classes.

Other utility classes in the project include `ProxyUtil`, `ModelCostCalculator`, `HttpException`, `HttpStatus`, `Compression`, `Base58`, `DoubleStringDeserializer`, `BufferingReadStream`, and `UrlUtil`. Each of these classes provides specific functionality that is used in different parts of the application.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/util/ResourceUtil.java" line="18">

---

# ResourceUtil class

This is the `ResourceUtil` class. It provides several static methods for working with resources, such as `hasResource`, `getResourceType`, `getBucket`, `sharedResourcesToMap`, and `resourceFromUrl`.

```java
@UtilityClass
public class ResourceUtil {

    public static boolean hasResource(ResourceDescription resource, ResourceService resourceService, BlobStorage storage) {
        return switch (resource.getType()) {
            case FILE -> storage.exists(resource.getAbsoluteFilePath());
            case CONVERSATION, PROMPT, APPLICATION -> resourceService.hasResource(resource);
            default -> throw new IllegalArgumentException("Unsupported resource type " + resource.getType());
        };
    }

    public ResourceType getResourceType(String url) {
        if (url == null) {
            throw new IllegalStateException("Resource link can not be null");
        }

        String[] paths = url.split(BlobStorageUtil.PATH_SEPARATOR);

        if (paths.length < 2) {
            throw new IllegalStateException("Invalid resource link provided: " + url);
        }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="70">

---

# Usage of ResourceUtil in ShareService

Here, `ResourceUtil` is used in the `ShareService` class. The `sharedResourcesToMap` method is used to convert a list of shared resources to a map. The `getResourceType` and `getBucket` methods are used to get the type and bucket of a resource, respectively.

```java
            if (sharedResources != null) {
                Map<String, Set<ResourceAccessType>> links = ResourceUtil.sharedResourcesToMap(sharedResources.getResources());
                resultMetadata.addAll(linksToMetadata(links));
            }
        }

        return new SharedResourcesResponse(resultMetadata);
    }

    /**
     * Returns list of resources shared by user.
     *
     * @param bucket - user bucket
     * @param location - storage location
     * @param request - request body
     * @return list of shared with user resources
     */
    public SharedResourcesResponse listSharedByMe(String bucket, String location, ListSharedResourcesRequest request) {
        Set<ResourceType> requestedResourceTypes = request.getResourceTypes();

        Set<ResourceDescription> shareResources = new HashSet<>();
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ShareController.java" line="140">

---

# Usage of ResourceUtil in ShareController

In the `ShareController` class, the `resourceFromUrl` method of `ResourceUtil` is used to create a `ResourceDescription` from a URL.

```java
                            .collect(Collectors.toUnmodifiableMap(
                                    resource -> ResourceUtil.resourceFromUrl(resource.url(), encryptionService),
                                    SharedResource::permissions));
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ResourceOperationService.java" line="57">

---

# Usage of ResourceUtil in ResourceOperationService

In the `ResourceOperationService` class, the `hasResource` method of `ResourceUtil` is used to check if a resource exists.

```java
    private boolean hasResource(ResourceDescription resource) {
        return ResourceUtil.hasResource(resource, resourceService, storage);
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/data/SharedResources.java" line="30">

---

# Usage of ResourceUtil in SharedResources

In the `SharedResources` class, the `sharedResourcesToMap` method of `ResourceUtil` is used to convert a list of shared resources to a map.

```java
    public void addSharedResources(Map<String, Set<ResourceAccessType>> sharedResources) {
        Map<String, Set<ResourceAccessType>> resourcesMap = ResourceUtil.sharedResourcesToMap(resources);
        sharedResources.forEach((url, permissions) -> {
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
