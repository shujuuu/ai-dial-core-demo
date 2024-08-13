---
title: Getting Started with Identity Management
---
The IdentityProvider class in the Security package is responsible for managing user identities in the application. It provides mechanisms for verifying user credentials, extracting user roles, and managing user sessions. The class uses JWT (JSON Web Tokens) for authentication and authorization purposes. It also provides a caching mechanism for storing the results obtained from the JWK (JSON Web Key) provider, which is used for cryptographic operations.

The IdentityProvider class uses a JwkProvider to handle JWKs (JSON Web Keys). JWKs are a format for expressing public key cryptography keys using JSON. They are used in the IdentityProvider class to verify the JWTs.

The IdentityProvider class also uses a Vertx instance and an HttpClient for handling HTTP requests and responses. Vertx is a toolkit for building reactive applications on the JVM. HttpClient is used to send HTTP requests and receive HTTP responses.

The IdentityProvider class has a method called extractClaimsFromJwt which is used to extract claims from a decoded JWT. Claims are pieces of information asserted about a subject and they are packaged in a JWT.

The IdentityProvider class also has a method called extractClaimsFromUserInfo which is used to extract claims from user info. This method sends a GET request to the userInfoUrl and extracts the claims from the response.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/security/IdentityProvider.java" line="38">

---

# IdentityProvider Class

This is the definition of the IdentityProvider class. It includes various fields for managing user identities, such as the JwkProvider for handling JWKs, the Vertx instance and HttpClient for handling HTTP requests and responses, and various settings for managing JWT verification and user roles.

```java
public class IdentityProvider {

    // path to the claim of user roles in JWT
    private final String[] rolePath;

    private JwkProvider jwkProvider;

    private URL userInfoUrl;

    // in memory cache store results obtained from JWK provider
    private final ConcurrentHashMap<String, Future<JwkResult>> cache = new ConcurrentHashMap<>();

    // the name of the claim in JWT to extract user email
    private final String loggingKey;
    // random salt is used to digest user email
    private final String loggingSalt;

    private final MessageDigest sha256Digest;

    // the flag determines if user email should be obfuscated
    private final boolean obfuscateUserEmail;
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/security/IdentityProvider.java" line="250">

---

# Extracting Claims from JWT

This is the `extractClaimsFromJwt` method. It is used to extract claims from a decoded JWT. Claims are pieces of information asserted about a subject and they are packaged in a JWT.

```java
    Future<ExtractedClaims> extractClaimsFromJwt(DecodedJWT decodedJwt) {
        if (decodedJwt == null) {
            return Future.failedFuture(new IllegalArgumentException("decoded JWT must not be null"));
        }
        if (disableJwtVerification) {
            return Future.succeededFuture(from(decodedJwt));
        }
        return verifyJwt(decodedJwt).map(this::from);
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/security/IdentityProvider.java" line="260">

---

# Extracting Claims from User Info

This is the `extractClaimsFromUserInfo` method. It is used to extract claims from user info. This method sends a GET request to the `userInfoUrl` and extracts the claims from the response.

```java
    Future<ExtractedClaims> extractClaimsFromUserInfo(String accessToken) {
        RequestOptions options = new RequestOptions()
                .setAbsoluteURI(userInfoUrl)
                .setMethod(HttpMethod.GET);
        Promise<ExtractedClaims> promise = Promise.promise();
        client.request(options).onFailure(promise::fail).onSuccess(request -> {
            request.putHeader(HttpHeaders.AUTHORIZATION, "Bearer " + accessToken);
            request.send().onFailure(promise::fail).onSuccess(response -> {
                if (response.statusCode() != 200) {
                    promise.fail(String.format("Request failed with http code %d", response.statusCode()));
                    return;
                }
                response.body().map(body -> {
                    try {
                        JsonObject json = body.toJsonObject();
                        from(accessToken, json, promise);
                    } catch (Throwable e) {
                        promise.fail(e);
                    }
                    return null;
                }).onFailure(promise::fail);
```

---

</SwmSnippet>

# IdentityProvider Functions

In this section, we will discuss the main functions of the IdentityProvider class, focusing on the methods for extracting claims from JWT and user info.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/security/IdentityProvider.java" line="250">

---

## Extracting Claims from JWT

The `extractClaimsFromJwt` method is used to extract claims from a decoded JWT. If JWT verification is disabled, it directly returns the claims from the decoded JWT. Otherwise, it verifies the JWT before extracting the claims.

```java
    Future<ExtractedClaims> extractClaimsFromJwt(DecodedJWT decodedJwt) {
        if (decodedJwt == null) {
            return Future.failedFuture(new IllegalArgumentException("decoded JWT must not be null"));
        }
        if (disableJwtVerification) {
            return Future.succeededFuture(from(decodedJwt));
        }
        return verifyJwt(decodedJwt).map(this::from);
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/security/IdentityProvider.java" line="260">

---

## Extracting Claims from User Info

The `extractClaimsFromUserInfo` method is used to extract claims from user info. It sends a GET request to the userInfoUrl and extracts the claims from the response.

```java
    Future<ExtractedClaims> extractClaimsFromUserInfo(String accessToken) {
        RequestOptions options = new RequestOptions()
                .setAbsoluteURI(userInfoUrl)
                .setMethod(HttpMethod.GET);
        Promise<ExtractedClaims> promise = Promise.promise();
        client.request(options).onFailure(promise::fail).onSuccess(request -> {
            request.putHeader(HttpHeaders.AUTHORIZATION, "Bearer " + accessToken);
            request.send().onFailure(promise::fail).onSuccess(response -> {
                if (response.statusCode() != 200) {
                    promise.fail(String.format("Request failed with http code %d", response.statusCode()));
                    return;
                }
                response.body().map(body -> {
                    try {
                        JsonObject json = body.toJsonObject();
                        from(accessToken, json, promise);
                    } catch (Throwable e) {
                        promise.fail(e);
                    }
                    return null;
                }).onFailure(promise::fail);
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
