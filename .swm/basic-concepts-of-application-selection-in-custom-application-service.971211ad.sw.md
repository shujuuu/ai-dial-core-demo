---
title: Basic Concepts of Application Selection in Custom Application Service
---
Proxy Application Selection in the Custom Application Service refers to the process of choosing a specific application to handle a request. This is done based on the URL provided in the request. The service checks if the URL is valid, if the resource type is an application, and if the user has read access to the application. If all these conditions are met, the application is selected and its body is fetched from the resource service. The body is then converted to an Application object and returned.

The Custom Application Service also provides methods to fetch lists of applications. These include methods to fetch a list of custom applications from the user's bucket, a list of custom applications shared with the user, and a list of published custom applications. These methods use the resource service to fetch the applications and the access service to check if the user has access to them.

The service also includes a method to check if custom applications should be included. This is determined by a configuration setting.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/CustomApplicationService.java" line="42">

---

# Loading a Custom Application

The `getCustomApplication` method is used to load a custom application from a provided URL. It first validates the URL and checks if the resource type is an application. Then, it verifies if the user has read access to the application. If all these checks pass, it fetches the application's body from the resource service and converts it into an Application object.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/CustomApplicationService.java" line="68">

---

# Fetching Lists of Applications

The `getOwnCustomApplications` method fetches a list of custom applications from the user's bucket. It uses the resource service to fetch the applications and the access service to check if the user has access to them.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/CustomApplicationService.java" line="83">

---

The `getSharedApplications` method fetches a list of custom applications shared with the user. It uses the resource service to fetch the applications and the access service to check if the user has access to them.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/CustomApplicationService.java" line="105">

---

The `getPublicApplications` method fetches a list of published custom applications. It uses the resource service to fetch the applications and the access service to check if the user has access to them.

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

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/CustomApplicationService.java" line="128">

---

# Checking if Custom Applications Should be Included

The `includeCustomApplications` method checks a configuration setting to determine if custom applications should be included.

```java
    public boolean includeCustomApplications() {
        return applicationsConfig.getBoolean("includeCustomApps", false);
    }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
