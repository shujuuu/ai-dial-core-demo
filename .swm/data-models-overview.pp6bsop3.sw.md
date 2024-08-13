---
title: Data Models Overview
---
Data Models in ai-dial-core-demo refer to the structure of data that is used and manipulated within the application. They are represented as classes in the `com.epam.aidial.core.data` package. These classes define the properties and behaviors of the data objects within the application.

One of the key data models in the application is the `ModelData` class. This class extends `DeploymentData` and includes properties such as `lifecycleStatus`, `capabilities`, `tokenizerModel`, `limits`, and `pricing`. These properties represent different aspects of a model in the application.

The `ModelData` class is used in various parts of the application. For instance, in the `ModelController` class, instances of `ModelData` are created and manipulated. This class is responsible for handling HTTP requests related to models.

Another important data model is the `FeaturesData` class. This class defines a set of boolean properties that represent different features that can be enabled or disabled in the application. The `FeaturesData` class is used in the `DeploymentController` class and the `DeploymentData` class.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/data/ModelData.java" line="9">

---

# ModelData Class

The `ModelData` class is a key data model in the application. It extends `DeploymentData` and includes properties such as `lifecycleStatus`, `capabilities`, `tokenizerModel`, `limits`, and `pricing`. These properties represent different aspects of a model in the application.

```java
@Data
@EqualsAndHashCode(callSuper = true)
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class ModelData extends DeploymentData {

    private String lifecycleStatus = "generally-available";
    private CapabilitiesData capabilities = new CapabilitiesData();
    private String tokenizerModel;
    private TokenLimitsData limits;
    private PricingData pricing;

    {
        setObject("model");
        setScaleSettings(null);
    }
}
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/ModelController.java" line="37">

---

# Usage of ModelData

`ModelData` is used in the `ModelController` class. Instances of `ModelData` are created and manipulated to handle HTTP requests related to models.

```java
        ModelData data = createModel(model);
        context.respond(HttpStatus.OK, data);
        return Future.succeededFuture();
    }

    public Future<?> getModels() {
        Config config = context.getConfig();
        List<ModelData> models = new ArrayList<>();

        for (Model model : config.getModels().values()) {
            if (DeploymentController.hasAccess(context, model)) {
                ModelData data = createModel(model);
                models.add(data);
            }
        }

        ListData<ModelData> list = new ListData<>();
        list.setData(models);
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/data/FeaturesData.java" line="8">

---

# FeaturesData Class

The `FeaturesData` class is another important data model. It defines a set of boolean properties that represent different features that can be enabled or disabled in the application.

```java
@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class FeaturesData {
    private boolean rate = false;
    private boolean tokenize = false;
    private boolean truncatePrompt = false;
    private boolean configuration = false;

    private boolean systemPrompt = true;
    private boolean tools = false;
    private boolean seed = false;
    private boolean urlAttachments = false;
    private boolean folderAttachments = false;
}
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/controller/DeploymentController.java" line="102">

---

# Usage of FeaturesData

`FeaturesData` is used in the `DeploymentController` class. Instances of `FeaturesData` are created to represent the features of a deployment.

```java
    static FeaturesData createFeatures(Features features) {
        FeaturesData data = new FeaturesData();
```

---

</SwmSnippet>

# Data Models Overview

Data models in ai-dial-core-demo are represented as classes that define the properties and behaviors of the data objects within the application.

<SwmSnippet path="/src/main/java/com/epam/aidial/core/data/ModelData.java" line="9">

---

## ModelData

The `ModelData` class extends `DeploymentData` and includes properties such as `lifecycleStatus`, `capabilities`, `tokenizerModel`, `limits`, and `pricing`. These properties represent different aspects of a model in the application.

```java
@Data
@EqualsAndHashCode(callSuper = true)
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class ModelData extends DeploymentData {

    private String lifecycleStatus = "generally-available";
    private CapabilitiesData capabilities = new CapabilitiesData();
    private String tokenizerModel;
    private TokenLimitsData limits;
    private PricingData pricing;

    {
        setObject("model");
        setScaleSettings(null);
    }
}
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/data/FeaturesData.java" line="8">

---

## FeaturesData

The `FeaturesData` class defines a set of boolean properties that represent different features that can be enabled or disabled in the application.

```java
@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class FeaturesData {
    private boolean rate = false;
    private boolean tokenize = false;
    private boolean truncatePrompt = false;
    private boolean configuration = false;

    private boolean systemPrompt = true;
    private boolean tools = false;
    private boolean seed = false;
    private boolean urlAttachments = false;
    private boolean folderAttachments = false;
}
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/data/CapabilitiesData.java" line="8">

---

## CapabilitiesData

The `CapabilitiesData` class includes properties that represent the capabilities of a model in the application.

```java
@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class CapabilitiesData {
    private String[] scaleTypes = {"standard"};
    private boolean completion;
    private boolean chatCompletion;
    private boolean embeddings;
    private boolean fineTune;
    private boolean inference;
}
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/data/ListData.java" line="10">

---

## ListData

The `ListData` class is a generic class that represents a list of data objects in the application.

```java
@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class ListData<T> {
    private List<T> data = List.of();
    private String object = "list";
}
```

---

</SwmSnippet>

<SwmSnippet path="/src/main/java/com/epam/aidial/core/data/ApplicationData.java" line="9">

---

## ApplicationData

The `ApplicationData` class extends `DeploymentData` and is used to represent an application in the ai-dial-core-demo application.

```java
@Data
@EqualsAndHashCode(callSuper = true)
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class ApplicationData extends DeploymentData {
    {
        setObject("application");
        setScaleSettings(null);
    }
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
