---
title: Security Overview
---
Security in ai-dial-core-demo refers to the measures taken to protect the application's data and ensure its integrity. It involves the use of encryption, identity verification, and access control mechanisms.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/security/EncryptionService.java" line="20">

---

For instance, the `EncryptionService` class uses the AES/CBC/PKCS5Padding transformation for encryption, which is a standard cryptographic algorithm for secure data transmission.

```java
    private static final String TRANSFORMATION = "AES/CBC/PKCS5Padding";
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/security/IdentityProvider.java" line="23">

---

The `IdentityProvider` class, on the other hand, uses the `MessageDigest` class from the `java.security` package to generate a unique hash for each user, ensuring that user identities are securely managed.

```java
import java.security.MessageDigest;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/security/EncryptionService.java" line="11">

---

The `SecretKey` class from the `javax.crypto` package is used to generate a secret cryptographic key for encryption and decryption processes.

```java
import javax.crypto.SecretKey;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/security/IdentityProvider.java" line="64">

---

# IdentityProvider Class

In the IdentityProvider class, the duration of how long the JWT results should be stored in the cache is specified. This is an example of how security measures are implemented in the application.

```java
    // the duration is how many milliseconds success JWK result should be stored in the cache
    private final long positiveCacheExpirationMs;

    // the duration is how many milliseconds failed JWK result should be stored in the cache
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
