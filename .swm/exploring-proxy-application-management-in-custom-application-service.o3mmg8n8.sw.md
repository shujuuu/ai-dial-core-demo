---
title: Exploring Proxy Application Management in Custom Application Service
---
Proxy Application Handling in the Custom Application Service refers to the management of custom applications through a proxy. This involves loading, fetching, and handling access permissions for custom applications. The service provides methods to get custom applications, own custom applications, shared applications, and public applications. Each of these methods handles applications differently based on their access permissions and source (own, shared, or public).

The service also includes methods to fetch applications recursively and individually. The recursive method is used to fetch all applications within a resource, while the individual fetch method is used to retrieve a single application. These methods are essential for managing the retrieval of applications based on their location and access permissions.

Additionally, the service provides a method to include custom applications. This method checks a configuration setting to determine whether custom applications should be included in the application handling process. This allows for flexible management of applications based on the system's configuration.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/CustomApplicationService.java" line="26">

---

# CustomApplicationService Class

The CustomApplicationService class is the main class responsible for Proxy Application Handling. It provides methods for loading, fetching, and managing access permissions for custom applications.

```java
public class CustomApplicationService {

    private static final int FETCH_SIZE = 1000;

    private final EncryptionService encryptionService;
    private final ResourceService resourceService;
    private final ShareService shareService;
    private final AccessService accessService;
    private final JsonObject applicationsConfig;

    /**
     * Loads a custom application from provided url with permission check
     *
     * @param url - custom application url
     * @return - custom application or null if invalid url provided or resource not found
     */
    public Application getCustomApplication(String url, ProxyContext context) {
        ResourceDescription resource;
        try {
            resource = ResourceDescription.fromAnyUrl(url, encryptionService);
        } catch (Exception e) {
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/CustomApplicationService.java" line="132">

---

# Fetching Applications

The fetchApplicationsRecursively and fetchApplication methods are used to retrieve applications. The recursive method fetches all applications within a resource, while the individual fetch method retrieves a single application.

```java
    private void fetchApplicationsRecursively(ResourceDescription description, List<Application> result) {
        if (!description.isFolder()) {
            Application application = fetchApplication(description.getUrl());
            if (application != null) {
                result.add(application);
            }
            return;
        }

        String nextToken = null;
        boolean fetchMore = true;
        while (fetchMore) {
            MetadataBase metadataResponse = resourceService.getMetadata(description, nextToken, FETCH_SIZE, true);
            // if no metadata present - stop fetching
            if (metadataResponse == null) {
                return;
            }

            if (metadataResponse instanceof ResourceFolderMetadata folderMetadata) {
                nextToken = folderMetadata.getNextToken();
                fetchMore = nextToken != null;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/service/CustomApplicationService.java" line="128">

---

# Including Custom Applications

The includeCustomApplications method checks a configuration setting to determine whether custom applications should be included in the application handling process. This allows for flexible management of applications based on the system's configuration.

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
