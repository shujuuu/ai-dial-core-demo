---
title: Introduction to Application Selection in Application Service
---
Proxy Application Selection refers to the process of choosing a specific application to handle a request in the Application Service. This is done through various methods in the CustomApplicationService class, such as getPublicApplications, getSharedApplications, getOwnCustomApplications, and getCustomApplication. These methods retrieve applications based on different criteria, such as whether they are public, shared, owned by the user, or specified by a custom URL. The selected application is then used to process the incoming request.

The getPublicApplications method retrieves a list of all public applications. It fetches the metadata for each application in the public bucket and filters out any applications that the user does not have access to. The resulting list of applications is then returned.

The getSharedApplications method retrieves a list of all applications that have been shared with the user. It fetches the metadata for each shared application and recursively fetches applications from each resource. The resulting list of applications is then returned.

The getOwnCustomApplications method retrieves a list of all applications owned by the user. It fetches the metadata for each application in the user's bucket and recursively fetches applications from each resource. The resulting list of applications is then returned.

The getCustomApplication method retrieves a specific application specified by a custom URL. It checks if the resource at the URL is of the type APPLICATION and if the user has read access to it. If both conditions are met, it fetches the application and returns it.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/CustomApplicationService.java" line="105">

---

# Public Applications

The `getPublicApplications` method retrieves a list of all public applications. It fetches the metadata for each application in the public bucket and filters out any applications that the user does not have access to. The resulting list of applications is then returned.

```java
    public List<Application> getPublicApplications(ProxyContext context) {
        List<Application> applications = new ArrayList<>();
        ResourceDescription rootFolderResource = ResourceDescription.fromDecoded(ResourceType.APPLICATION, BlobStorageUtil.PUBLIC_BUCKET, BlobStorageUtil.PUBLIC_LOCATION, null);

        String nextToken = null;
        boolean fetchMore = true;
        while (fetchMore) {
            ResourceFolderMetadata metadataResponse = resourceService.getFolderMetadata(rootFolderResource, nextToken, FETCH_SIZE, true);
            // if no metadata present - stop fetching
            if (metadataResponse == null) {
                return applications;
            }

            nextToken = metadataResponse.getNextToken();
            fetchMore = nextToken != null;

            accessService.filterForbidden(context, rootFolderResource, metadataResponse);
            fetchItems(metadataResponse.getItems(), applications);
        }

        return applications;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/CustomApplicationService.java" line="83">

---

# Shared Applications

The `getSharedApplications` method retrieves a list of all applications that have been shared with the user. It fetches the metadata for each shared application and recursively fetches applications from each resource. The resulting list of applications is then returned.

```java
    public List<Application> getSharedApplications(ProxyContext context) {
        String bucketLocation = BlobStorageUtil.buildInitiatorBucket(context);
        String bucket = encryptionService.encrypt(bucketLocation);

        ListSharedResourcesRequest listSharedResourcesRequest = new ListSharedResourcesRequest();
        listSharedResourcesRequest.setResourceTypes(Set.of(ResourceType.APPLICATION));
        SharedResourcesResponse sharedResourcesResponse = shareService.listSharedWithMe(bucket, bucketLocation, listSharedResourcesRequest);
        Set<MetadataBase> metadata = sharedResourcesResponse.getResources();
        // metadata can be either item or folder
        List<Application> applications = new ArrayList<>();
        for (MetadataBase meta : metadata) {
            ResourceDescription resource = ResourceDescription.fromAnyUrl(meta.getUrl(), encryptionService);
            fetchApplicationsRecursively(resource, applications);
        }

        return applications;
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/CustomApplicationService.java" line="68">

---

# User's Custom Applications

The `getOwnCustomApplications` method retrieves a list of all applications owned by the user. It fetches the metadata for each application in the user's bucket and recursively fetches applications from each resource. The resulting list of applications is then returned.

```java
    public List<Application> getOwnCustomApplications(ProxyContext context) {
        String bucketLocation = BlobStorageUtil.buildInitiatorBucket(context);
        String bucket = encryptionService.encrypt(bucketLocation);

        List<Application> applications = new ArrayList<>();
        ResourceDescription rootFolderResource = ResourceDescription.fromDecoded(ResourceType.APPLICATION, bucket, bucketLocation, null);
        fetchApplicationsRecursively(rootFolderResource, applications);

        return applications;
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/CustomApplicationService.java" line="42">

---

# Custom Application by URL

The `getCustomApplication` method retrieves a specific application specified by a custom URL. It checks if the resource at the URL is of the type APPLICATION and if the user has read access to it. If both conditions are met, it fetches the application and returns it.

```java
    public Application getCustomApplication(String url, ProxyContext context) {
        ResourceDescription resource;
        try {
            resource = ResourceDescription.fromAnyUrl(url, encryptionService);
        } catch (Exception e) {
            log.warn("Invalid resource url provided: {}", url);
            throw new ResourceNotFoundException("Application %s not found".formatted(url));
        }

        if (resource.getType() != ResourceType.APPLICATION) {
            throw new IllegalArgumentException("Unsupported deployment type: " + resource.getType());
        }

        boolean hasAccess = accessService.hasReadAccess(resource, context);
        if (!hasAccess) {
            throw new PermissionDeniedException("User don't have access to the deployment " + url);
        }

        String applicationBody = resourceService.getResource(resource);
        return ProxyUtil.convertToObject(applicationBody, Application.class, true);
    }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
