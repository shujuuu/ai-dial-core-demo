---
title: Basic Concepts of Resource Sharing Service
---
The ShareService is a crucial component in the service layer of the application. It is responsible for managing the sharing of resources between users. This includes functionalities such as listing resources shared with or by a user, initializing share requests, accepting shared resources, and revoking shared resources. The service also provides methods to manage permissions for shared resources.

The ShareService uses several other services to perform its tasks. These include the ResourceService, InvitationService, EncryptionService, and BlobStorage. The ResourceService is used to interact with resources, the InvitationService is used to manage invitations for sharing, the EncryptionService is used for encrypting user locations, and the BlobStorage is used for storing blobs.

The ShareService class contains several methods that perform specific tasks. For example, the 'listSharedWithMe' method returns a list of resources shared with a user, the 'listSharedByMe' method returns a list of resources shared by a user, the 'initializeShare' method initializes a share request by creating an invitation object, and the 'acceptSharedResources' method allows a user to accept an invitation to grant share access for provided resources.

The ShareService also contains methods to manage permissions for shared resources. The 'revokeSharedResource' method revokes share access for a provided resource, the 'revokeSharedAccess' method revokes share access for provided resources, and the 'addSharedResource' method adds a shared resource with specified permissions.

In addition to these, the ShareService contains several helper methods that are used internally to perform various tasks. These include methods to remove shared resource permissions, get share resources, and check if a resource exists.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="38">

---

# ShareService Class

The ShareService class is defined here. It has several dependencies including ResourceService, InvitationService, EncryptionService, and BlobStorage. These dependencies are used to interact with resources, manage invitations for sharing, encrypt user locations, and store blobs respectively.

```java
@Slf4j
@AllArgsConstructor
public class ShareService {

    private static final String SHARE_RESOURCE_FILENAME = "share";

    private final ResourceService resourceService;
    private final InvitationService invitationService;
    private final EncryptionService encryptionService;
    private final BlobStorage storage;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="57">

---

# Listing Shared Resources

The 'listSharedWithMe' method is used to return a list of resources shared with a user. It takes the user's bucket, location, and a request body as parameters and returns a list of shared resources.

```java
    public SharedResourcesResponse listSharedWithMe(String bucket, String location, ListSharedResourcesRequest request) {
        Set<ResourceType> requestedResourceType = request.getResourceTypes();

        Set<ResourceDescription> shareResources = new HashSet<>();
        for (ResourceType resourceType : requestedResourceType) {
            ResourceDescription sharedResource = getShareResource(ResourceType.SHARED_WITH_ME, resourceType, bucket, location);
            shareResources.add(sharedResource);
        }

        Set<MetadataBase> resultMetadata = new HashSet<>();
        for (ResourceDescription resource : shareResources) {
            String sharedResource = resourceService.getResource(resource);
            SharedResources sharedResources = ProxyUtil.convertToObject(sharedResource, SharedResources.class);
            if (sharedResources != null) {
                Map<String, Set<ResourceAccessType>> links = ResourceUtil.sharedResourcesToMap(sharedResources.getResources());
                resultMetadata.addAll(linksToMetadata(links));
            }
        }

        return new SharedResourcesResponse(resultMetadata);
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="117">

---

# Initializing Share Requests

The 'initializeShare' method is used to initialize a share request by creating an invitation object. It validates the resources and creates an invitation using the InvitationService.

```java
    public InvitationLink initializeShare(String bucket, String location, ShareResourcesRequest request) {
        // validate resources - owner must be current user
        Set<SharedResource> sharedResources = request.getResources();
        if (sharedResources.isEmpty()) {
            throw new IllegalArgumentException("No resources provided");
        }

        Set<String> uniqueLinks = new HashSet<>();
        List<SharedResource> normalizedResourceLinks = new ArrayList<>(sharedResources.size());
        for (SharedResource sharedResource : sharedResources) {
            ResourceDescription resource = getResourceFromLink(sharedResource.url());
            if (!bucket.equals(resource.getBucketName())) {
                throw new IllegalArgumentException("Resource %s does not belong to the user".formatted(resource.getUrl()));
            }
            if (!uniqueLinks.add(resource.getUrl())) {
                throw new IllegalArgumentException("Duplicated resource: %s".formatted(resource.getUrl()));
            }
            normalizedResourceLinks.add(sharedResource.withUrl(resource.getUrl()));
        }

        Invitation invitation = invitationService.createInvitation(bucket, location, normalizedResourceLinks);
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="148">

---

# Accepting Shared Resources

The 'acceptSharedResources' method is used to accept an invitation to grant share access for provided resources. It retrieves the invitation using the InvitationService and then adds the shared resources to the user's resources.

```java
    public void acceptSharedResources(String bucket, String location, String invitationId) {
        Invitation invitation = invitationService.getInvitation(invitationId);

        if (invitation == null) {
            throw new ResourceNotFoundException("No invitation found with id: " + invitationId);
        }

        List<SharedResource> resourceLinks = invitation.getResources();
        for (SharedResource link : resourceLinks) {
            String url = link.url();
            if (ResourceDescription.fromPrivateUrl(url, encryptionService).getBucketName().equals(bucket)) {
                throw new IllegalArgumentException("Resource %s already belong to you".formatted(url));
            }
        }

        // group resources with the same type to reduce resource transformations
        Map<ResourceType, List<SharedResource>> resourceGroups = resourceLinks.stream()
                .collect(Collectors.groupingBy(sharedResource -> ResourceUtil.getResourceType(sharedResource.url())));

        resourceGroups.forEach((resourceType, links) -> {
            String ownerBucket = ResourceUtil.getBucket(links.get(0).url());
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="254">

---

# Revoking Shared Resources

The 'revokeSharedResource' method is used to revoke share access for a provided resource. It calls the 'revokeSharedAccess' method to perform the revocation.

```java
    public void revokeSharedResource(
            String bucket, String location, ResourceDescription resourceLink) {
        revokeSharedAccess(bucket, location, Map.of(resourceLink, ResourceAccessType.ALL));
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="391">

---

# Helper Methods

The 'removeSharedResourcePermissions' method is a helper method used to remove permissions from a shared resource. It retrieves the shared resource, computes the new permissions, and updates the resource.

```java
    private void removeSharedResourcePermissions(
            String bucket, String location, String link, ResourceType resourceType, Set<ResourceAccessType> permissionsToRemove) {
        ResourceDescription sharedByMeResource = getShareResource(ResourceType.SHARED_WITH_ME, resourceType, bucket, location);
        resourceService.computeResource(sharedByMeResource, state -> {
            SharedResources sharedWithMe = ProxyUtil.convertToObject(state, SharedResources.class);
            if (sharedWithMe != null) {
                Set<ResourceAccessType> permissions = EnumSet.noneOf(ResourceAccessType.class);
                permissions.addAll(sharedWithMe.findPermissions(link));
                permissions.removeAll(permissionsToRemove);
                sharedWithMe.getResources().removeIf(resource -> link.equals(resource.url()));
                if (!permissions.isEmpty()) {
                    sharedWithMe.getResources().add(new SharedResource(link, permissions));
                }
            }

            return ProxyUtil.convertToString(sharedWithMe);
        });
    }
```

---

</SwmSnippet>

# ShareService Functions

This section provides an overview of the main functions in the ShareService class.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="57">

---

## listSharedWithMe

The `listSharedWithMe` method returns a list of resources shared with a user. It takes the user's bucket, storage location, and a request body as parameters. The method retrieves the types of resources requested, gets the shared resources, and then retrieves and converts the shared resources to a SharedResources object. Finally, it returns a response containing the metadata of the shared resources.

```java
    public SharedResourcesResponse listSharedWithMe(String bucket, String location, ListSharedResourcesRequest request) {
        Set<ResourceType> requestedResourceType = request.getResourceTypes();

        Set<ResourceDescription> shareResources = new HashSet<>();
        for (ResourceType resourceType : requestedResourceType) {
            ResourceDescription sharedResource = getShareResource(ResourceType.SHARED_WITH_ME, resourceType, bucket, location);
            shareResources.add(sharedResource);
        }

        Set<MetadataBase> resultMetadata = new HashSet<>();
        for (ResourceDescription resource : shareResources) {
            String sharedResource = resourceService.getResource(resource);
            SharedResources sharedResources = ProxyUtil.convertToObject(sharedResource, SharedResources.class);
            if (sharedResources != null) {
                Map<String, Set<ResourceAccessType>> links = ResourceUtil.sharedResourcesToMap(sharedResources.getResources());
                resultMetadata.addAll(linksToMetadata(links));
            }
        }

        return new SharedResourcesResponse(resultMetadata);
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="87">

---

## listSharedByMe

The `listSharedByMe` method returns a list of resources shared by a user. Similar to `listSharedWithMe`, it takes the user's bucket, storage location, and a request body as parameters. The method retrieves the types of resources requested, gets the shared resources, and then retrieves and converts the shared resources to a SharedByMeDto object. Finally, it returns a response containing the metadata of the shared resources.

```java
    public SharedResourcesResponse listSharedByMe(String bucket, String location, ListSharedResourcesRequest request) {
        Set<ResourceType> requestedResourceTypes = request.getResourceTypes();

        Set<ResourceDescription> shareResources = new HashSet<>();
        for (ResourceType resourceType : requestedResourceTypes) {
            ResourceDescription shareResource = getShareResource(ResourceType.SHARED_BY_ME, resourceType, bucket, location);
            shareResources.add(shareResource);
        }

        Set<MetadataBase> resultMetadata = new HashSet<>();
        for (ResourceDescription resource : shareResources) {
            String sharedResource = resourceService.getResource(resource);
            SharedByMeDto resourceToUsers = ProxyUtil.convertToObject(sharedResource, SharedByMeDto.class);
            if (resourceToUsers != null) {
                Map<String, Set<ResourceAccessType>> links = resourceToUsers.getAggregatedPermissions();
                resultMetadata.addAll(linksToMetadata(links));
            }
        }

        return new SharedResourcesResponse(resultMetadata);
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="117">

---

## initializeShare

The `initializeShare` method initializes a share request by creating an invitation object. It validates the resources to be shared, normalizes the resource links, and creates an invitation. The method returns an invitation link.

```java
    public InvitationLink initializeShare(String bucket, String location, ShareResourcesRequest request) {
        // validate resources - owner must be current user
        Set<SharedResource> sharedResources = request.getResources();
        if (sharedResources.isEmpty()) {
            throw new IllegalArgumentException("No resources provided");
        }

        Set<String> uniqueLinks = new HashSet<>();
        List<SharedResource> normalizedResourceLinks = new ArrayList<>(sharedResources.size());
        for (SharedResource sharedResource : sharedResources) {
            ResourceDescription resource = getResourceFromLink(sharedResource.url());
            if (!bucket.equals(resource.getBucketName())) {
                throw new IllegalArgumentException("Resource %s does not belong to the user".formatted(resource.getUrl()));
            }
            if (!uniqueLinks.add(resource.getUrl())) {
                throw new IllegalArgumentException("Duplicated resource: %s".formatted(resource.getUrl()));
            }
            normalizedResourceLinks.add(sharedResource.withUrl(resource.getUrl()));
        }

        Invitation invitation = invitationService.createInvitation(bucket, location, normalizedResourceLinks);
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="148">

---

## acceptSharedResources

The `acceptSharedResources` method allows a user to accept an invitation to grant share access for provided resources. It retrieves the invitation, validates the resource links, groups the resources by type, and then adds the user location to the resource owner and the links to the user.

```java
    public void acceptSharedResources(String bucket, String location, String invitationId) {
        Invitation invitation = invitationService.getInvitation(invitationId);

        if (invitation == null) {
            throw new ResourceNotFoundException("No invitation found with id: " + invitationId);
        }

        List<SharedResource> resourceLinks = invitation.getResources();
        for (SharedResource link : resourceLinks) {
            String url = link.url();
            if (ResourceDescription.fromPrivateUrl(url, encryptionService).getBucketName().equals(bucket)) {
                throw new IllegalArgumentException("Resource %s already belong to you".formatted(url));
            }
        }

        // group resources with the same type to reduce resource transformations
        Map<ResourceType, List<SharedResource>> resourceGroups = resourceLinks.stream()
                .collect(Collectors.groupingBy(sharedResource -> ResourceUtil.getResourceType(sharedResource.url())));

        resourceGroups.forEach((resourceType, links) -> {
            String ownerBucket = ResourceUtil.getBucket(links.get(0).url());
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="254">

---

## revokeSharedResource

The `revokeSharedResource` method revokes share access for a provided resource. It calls the `revokeSharedAccess` method with the resource and all access types.

```java
    public void revokeSharedResource(
            String bucket, String location, ResourceDescription resourceLink) {
        revokeSharedAccess(bucket, location, Map.of(resourceLink, ResourceAccessType.ALL));
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="266">

---

## revokeSharedAccess

The `revokeSharedAccess` method revokes share access for provided resources. It validates that all resources belong to the user performing the action, then for each resource, it removes the user locations and updates the resource.

```java
    public void revokeSharedAccess(
            String bucket, String location, Map<ResourceDescription, Set<ResourceAccessType>> permissionsToRevoke) {
        if (permissionsToRevoke.isEmpty()) {
            throw new IllegalArgumentException("No resources provided");
        }

        // validate that all resources belong to the user, who perform this action
        permissionsToRevoke.forEach((resource, permissions) -> {
            if (!resource.getBucketName().equals(bucket)) {
                throw new IllegalArgumentException("You are only allowed to revoke access from own resources");
            }
        });

        permissionsToRevoke.forEach((resource, permissionsToRemove) -> {
            ResourceType resourceType = resource.getType();
            String resourceUrl = resource.getUrl();
            ResourceDescription sharedByMeResource = getShareResource(ResourceType.SHARED_BY_ME, resourceType, bucket, location);
            String state = resourceService.getResource(sharedByMeResource);
            SharedByMeDto dto = ProxyUtil.convertToObject(state, SharedByMeDto.class);
            if (dto != null) {
                Set<String> userLocations = dto.collectUsersForPermissions(resourceUrl, permissionsToRemove);
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/ShareService.java" line="410">

---

## addSharedResource

The `addSharedResource` method adds a shared resource with specified permissions. It retrieves the shared resource, computes the resource, and then adds the shared resource with the specified permissions.

```java
    private void addSharedResource(
            String bucket,
            String location,
            String link,
            ResourceType resourceType,
            Set<ResourceAccessType> permissionsToAdd) {
        ResourceDescription sharedByMeResource = getShareResource(ResourceType.SHARED_WITH_ME, resourceType, bucket, location);
        resourceService.computeResource(sharedByMeResource, state -> {
            SharedResources sharedWithMe = ProxyUtil.convertToObject(state, SharedResources.class);
            if (sharedWithMe == null) {
                sharedWithMe = new SharedResources(new ArrayList<>());
            }
            Set<ResourceAccessType> permissions = EnumSet.noneOf(ResourceAccessType.class);
            permissions.addAll(sharedWithMe.findPermissions(link));
            permissions.addAll(permissionsToAdd);
            sharedWithMe.getResources().removeIf(resource -> link.equals(resource.url()));
            sharedWithMe.getResources().add(new SharedResource(link, permissions));

            return ProxyUtil.convertToString(sharedWithMe);
        });
    }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
