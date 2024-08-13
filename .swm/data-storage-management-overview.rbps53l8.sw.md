---
title: Data Storage Management Overview
---
Storage Management in ai-dial-core-demo refers to the functionality of managing data storage across different platforms. It is implemented through the BlobStorage class, which provides methods for storing, loading, and checking the existence of data in the storage.

The BlobStorage class uses the StorageProvider enum to determine the type of storage provider to use. The StorageProvider enum includes options for AWS S3, Google Cloud Storage, Azure Blob, and local filesystem.

The BlobStorage class also uses the BlobStorageUtil class, which provides utility methods for handling storage paths and content types. It includes methods for building user and app data buckets, determining if a path is a folder, and converting to storage paths.

The BlobStorage class is used throughout the codebase, including in the ResourceService, AiDial, BlobWriteStream, Proxy, and ResourceUtil classes, among others. This indicates that storage management is a crucial part of the application's functionality.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/storage/StorageProvider.java" line="3">

---

# StorageProvider Enum

The StorageProvider enum is used to specify the type of storage provider to use. It includes options for AWS S3, Google Cloud Storage, Azure Blob, and local filesystem.

```java
public enum StorageProvider {
    S3, AWS_S3, FILESYSTEM, GOOGLE_CLOUD_STORAGE, AZURE_BLOB;

    public static StorageProvider from(String storageProviderName) {
        return switch (storageProviderName) {
            case "s3" -> S3;
            case "aws-s3" -> AWS_S3;
            case "azureblob" -> AZURE_BLOB;
            case "google-cloud-storage" -> GOOGLE_CLOUD_STORAGE;
            case "filesystem" -> FILESYSTEM;
            default -> throw new IllegalArgumentException("Unknown storage provider");
        };
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/storage/BlobStorage.java" line="43">

---

# BlobStorage Class

The BlobStorage class provides methods for storing, loading, and checking the existence of data in the storage. It uses the StorageProvider enum to determine the type of storage provider to use.

```java
public class BlobStorage implements Closeable {

    // S3 implementation do not return a blob content type without additional head request.
    // To avoid additional request for each blob in the listing we try to recognize blob content type by its extension.
    // Default value is binary/octet-stream, see org.jclouds.s3.domain.ObjectMetadataBuilder
    private static final String DEFAULT_CONTENT_TYPE = ObjectMetadataBuilder.create().build().getContentMetadata().getContentType();

    private final BlobStoreContext storeContext;
    private final BlobStore blobStore;
    private final String bucketName;

    // defines a root folder for all resources in bucket
    @Getter
    @Nullable
    private final String prefix;

    public BlobStorage(Storage config) {
        String provider = config.getProvider();
        ContextBuilder builder = ContextBuilder.newBuilder(provider);
        if (config.getEndpoint() != null) {
            builder.endpoint(config.getEndpoint());
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/storage/BlobStorageUtil.java" line="9">

---

# BlobStorageUtil Class

The BlobStorageUtil class provides utility methods for handling storage paths and content types. It includes methods for building user and app data buckets, determining if a path is a folder, and converting to storage paths.

```java
@UtilityClass
public class BlobStorageUtil {

    public static final String PATH_SEPARATOR = "/";

    public static final String PUBLIC_BUCKET = "public";
    public static final String PUBLIC_LOCATION = PUBLIC_BUCKET + PATH_SEPARATOR;

    public static final String APPDATA_PATTERN = "appdata/%s";
    private static final String USER_BUCKET_PATTERN = "Users/%s/";
    private static final String API_KEY_BUCKET_PATTERN = "Keys/%s/";

    public String getContentType(String fileName) {
        String mimeType = MimeMapping.getMimeTypeForFilename(fileName);
        return mimeType == null ? "application/octet-stream" : mimeType;
    }

    public String buildUserBucket(ProxyContext context) {
        if (context.getApiKeyData().getPerRequestKey() == null) {
            return buildInitiatorBucket(context);
        } else {
```

---

</SwmSnippet>

# Storage Management Functions

This section will cover the main functions related to Storage Management in the ai-dial-core-demo project.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/storage/BlobStorage.java" line="43">

---

## BlobStorage

The BlobStorage class is responsible for managing data storage. It provides methods for storing, loading, and checking the existence of data in the storage. It uses the StorageProvider enum to determine the type of storage provider to use and the BlobStorageUtil class for handling storage paths and content types.

```java
public class BlobStorage implements Closeable {

    // S3 implementation do not return a blob content type without additional head request.
    // To avoid additional request for each blob in the listing we try to recognize blob content type by its extension.
    // Default value is binary/octet-stream, see org.jclouds.s3.domain.ObjectMetadataBuilder
    private static final String DEFAULT_CONTENT_TYPE = ObjectMetadataBuilder.create().build().getContentMetadata().getContentType();

    private final BlobStoreContext storeContext;
    private final BlobStore blobStore;
    private final String bucketName;

    // defines a root folder for all resources in bucket
    @Getter
    @Nullable
    private final String prefix;

    public BlobStorage(Storage config) {
        String provider = config.getProvider();
        ContextBuilder builder = ContextBuilder.newBuilder(provider);
        if (config.getEndpoint() != null) {
            builder.endpoint(config.getEndpoint());
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/storage/StorageProvider.java" line="3">

---

## StorageProvider

The StorageProvider enum includes options for AWS S3, Google Cloud Storage, Azure Blob, and local filesystem. It is used by the BlobStorage class to determine the type of storage provider to use.

```java
public enum StorageProvider {
    S3, AWS_S3, FILESYSTEM, GOOGLE_CLOUD_STORAGE, AZURE_BLOB;

    public static StorageProvider from(String storageProviderName) {
        return switch (storageProviderName) {
            case "s3" -> S3;
            case "aws-s3" -> AWS_S3;
            case "azureblob" -> AZURE_BLOB;
            case "google-cloud-storage" -> GOOGLE_CLOUD_STORAGE;
            case "filesystem" -> FILESYSTEM;
            default -> throw new IllegalArgumentException("Unknown storage provider");
        };
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/storage/BlobStorageUtil.java" line="10">

---

## BlobStorageUtil

The BlobStorageUtil class provides utility methods for handling storage paths and content types. It includes methods for building user and app data buckets, determining if a path is a folder, and converting to storage paths. It is used by the BlobStorage class.

```java
public class BlobStorageUtil {

    public static final String PATH_SEPARATOR = "/";

    public static final String PUBLIC_BUCKET = "public";
    public static final String PUBLIC_LOCATION = PUBLIC_BUCKET + PATH_SEPARATOR;

    public static final String APPDATA_PATTERN = "appdata/%s";
    private static final String USER_BUCKET_PATTERN = "Users/%s/";
    private static final String API_KEY_BUCKET_PATTERN = "Keys/%s/";

    public String getContentType(String fileName) {
        String mimeType = MimeMapping.getMimeTypeForFilename(fileName);
        return mimeType == null ? "application/octet-stream" : mimeType;
    }

    public String buildUserBucket(ProxyContext context) {
        if (context.getApiKeyData().getPerRequestKey() == null) {
            return buildInitiatorBucket(context);
        } else {
            return API_KEY_BUCKET_PATTERN.formatted(context.getSourceDeployment());
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
