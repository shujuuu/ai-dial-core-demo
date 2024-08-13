---
title: Getting started with Publication Management Service
---
The Publication Service is a crucial component of the service layer in the ai-dial-core-demo project. It is responsible for managing the lifecycle of publications, which are collections of resources that can be shared and accessed by users. The service provides functionalities such as creating, listing, approving, and rejecting publications. It also handles the validation of publication requests and ensures that the user has the necessary permissions to perform certain actions.

The Publication Service interacts with other services such as the Encryption Service, Resource Service, Access Service, Rule Service, and Notification Service. For instance, it uses the Encryption Service to encrypt bucket locations, the Resource Service to manage resources, and the Notification Service to send notifications to users.

The service also uses a BlobStorage instance for storing files and a Supplier for generating unique identifiers. It also uses a LongSupplier, which is a functional interface that supplies long-valued results, for generating timestamps.

The Publication Service class contains several methods that perform specific tasks. For example, the `approvePublication` method is used to approve a publication, the `getPublication` method is used to retrieve a specific publication, and the `createPublication` method is used to create a new publication. Each of these methods performs various checks and operations to ensure the integrity and correctness of the data.

The Publication Service also defines several constants that are used throughout its methods. For example, `PUBLICATIONS_NAME` is a constant that represents the name of the publications, and `ALLOWED_RESOURCES` is a set of resource types that are allowed in the publications.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/PublicationService.java" line="42">

---

# Publication Service Class

The Publication Service class contains several fields that are used to perform its functions. These include instances of other services like EncryptionService, ResourceService, AccessService, RuleService, and NotificationService. It also uses a BlobStorage instance for storing files and a Supplier for generating unique identifiers. It also uses a LongSupplier, which is a functional interface that supplies long-valued results, for generating timestamps.

```java
@RequiredArgsConstructor
public class PublicationService {

    private static final String PUBLICATIONS_NAME = "publications";

    private static final TypeReference<Map<String, Publication>> PUBLICATIONS_TYPE = new TypeReference<>() {
    };

    private static final ResourceDescription PUBLIC_PUBLICATIONS = ResourceDescription.fromDecoded(
            ResourceType.PUBLICATION, PUBLIC_BUCKET, PUBLIC_LOCATION, PUBLICATIONS_NAME);

    private static final Set<ResourceType> ALLOWED_RESOURCES = Set.of(ResourceType.FILE, ResourceType.CONVERSATION,
            ResourceType.PROMPT, ResourceType.APPLICATION);

    private final EncryptionService encryption;
    private final ResourceService resources;
    private final AccessService accessService;
    private final RuleService ruleService;
    private final NotificationService notificationService;
    private final BlobStorage files;
    private final Supplier<String> ids;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/PublicationService.java" line="217">

---

# Approve Publication Method

The `approvePublication` method is used to approve a publication. It performs various checks and operations to ensure the integrity and correctness of the data.

```java
    @Nullable
    public Publication approvePublication(ResourceDescription resource) {
        Publication publication = getPublication(resource);
        if (publication.getStatus() != Publication.Status.PENDING) {
            throw new ResourceNotFoundException("Publication is already finalized: " + resource.getUrl());
        }

        List<Publication.Resource> resourcesToAdd = publication.getResources().stream()
                .filter(i -> i.getAction() == Publication.ResourceAction.ADD)
                .toList();

        List<Publication.Resource> resourcesToDelete = publication.getResources().stream()
                .filter(i -> i.getAction() == Publication.ResourceAction.DELETE)
                .toList();

        checkReviewResources(resourcesToAdd);
        checkTargetResources(resourcesToAdd, false);
        checkTargetResources(resourcesToDelete, true);

        resources.computeResource(publications(resource), body -> {
            Map<String, Publication> publications = decodePublications(body);
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/PublicationService.java" line="127">

---

# Get Publication Method

The `getPublication` method is used to retrieve a specific publication. It performs various checks and operations to ensure the integrity and correctness of the data.

```java
    public Publication getPublication(ResourceDescription resource) {
        if (resource.getType() != ResourceType.PUBLICATION || resource.isPublic() || resource.isFolder() || resource.getParentPath() != null) {
            throw new IllegalArgumentException("Bad publication url: " + resource.getUrl());
        }

        ResourceDescription key = publications(resource);
        Map<String, Publication> publications = decodePublications(resources.getResource(key));
        Publication publication = publications.get(resource.getUrl());

        if (publication == null) {
            throw new ResourceNotFoundException("No publication: " + resource.getUrl());
        }

        return publication;
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/PublicationService.java" line="143">

---

# Create Publication Method

The `createPublication` method is used to create a new publication. It performs various checks and operations to ensure the integrity and correctness of the data.

```java
    public Publication createPublication(ProxyContext context, Publication publication) {
        String bucketLocation = BlobStorageUtil.buildInitiatorBucket(context);
        String bucket = encryption.encrypt(bucketLocation);

        prepareAndValidatePublicationRequest(context, bucket, bucketLocation, publication);

        List<Publication.Resource> resourcesToAdd = publication.getResources().stream()
                .filter(resource -> resource.getAction() == Publication.ResourceAction.ADD)
                .toList();

        // copy resources as is
        copySourceToReviewResources(resourcesToAdd);
        // replace links
        replaceSourceToReviewLinks(resourcesToAdd);

        resources.computeResource(publications(bucket, bucketLocation), body -> {
            Map<String, Publication> publications = decodePublications(body);

            if (publications.put(publication.getUrl(), publication) != null) {
                throw new IllegalStateException("Publication with such url already exists: " + publication.getUrl());
            }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/PublicationService.java" line="45">

---

# Constants in Publication Service

The Publication Service also defines several constants that are used throughout its methods. For example, `PUBLICATIONS_NAME` is a constant that represents the name of the publications, and `ALLOWED_RESOURCES` is a set of resource types that are allowed in the publications.

```java
    private static final String PUBLICATIONS_NAME = "publications";

    private static final TypeReference<Map<String, Publication>> PUBLICATIONS_TYPE = new TypeReference<>() {
    };

    private static final ResourceDescription PUBLIC_PUBLICATIONS = ResourceDescription.fromDecoded(
            ResourceType.PUBLICATION, PUBLIC_BUCKET, PUBLIC_LOCATION, PUBLICATIONS_NAME);

    private static final Set<ResourceType> ALLOWED_RESOURCES = Set.of(ResourceType.FILE, ResourceType.CONVERSATION,
            ResourceType.PROMPT, ResourceType.APPLICATION);
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
