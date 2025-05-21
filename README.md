# IIIF Embeddings Extension Proposal

## 1. Introduction

This document describes a way to store embedding vectors for IIIF resources in an Embedding Annotation. Embedding Annotations can be used to enable semantic search and understanding of the content within IIIF resources.

The IIIF Presentation API has the capability to support complex Web Annotations which can provide detailed and specific information regarding IIIF resources. You can see various use cases which implement such Web Annotations in the IIIF Cookbook. Building on the principles of extending Web Annotation for specific purposes, this document outlines a standard for incorporating embedding vectors.

### 1.1 Objectives and Scope

This document will supply vocabulary and a JSON-LD 1.1 context allowing for a JSON-LD pattern by which to extend Web Annotation and the IIIF Presentation API to support embedding vectors. This pattern promotes interoperability for semantic search and analysis across different IIIF platforms.

This extension supports both embedded and externally referenced embedding vectors, providing implementers with flexibility in handling vector datasets.

Further, the following use cases are in scope:

*   **Semantic Search:** Enabling search based on the meaning or content of regions within IIIF resources, rather than just keyword matching.
*   **Content Similarity:** Identifying visually or semantically similar regions across different IIIF resources.
*   **Content Recommendation:** Suggesting related IIIF resources based on semantic similarity.

The following use cases are not in scope:

*   **Specific Embedding Models:** This extension does not mandate the use of any particular embedding model or algorithm (e.g., CLIP, Word2Vec). The focus is on a general framework for representing and linking embedding vectors.
*   **Dimensionality Reduction Techniques:**  Specific methods for reducing the dimensionality of embedding vectors are not covered.
*   **Vector Search Implementation Details:**  This extension does not specify how vector search indices should be built or queried.

### 1.2 Motivating Use Cases

*   **Finding Images Similar to a Drawn Region:** A user draws a rough shape on an image and searches for other images containing similar visual content based on their embeddings.
*   **Searching for Documents Based on Thematic Similarity:** A researcher wants to find historical maps that discuss similar topics or depict similar geographical features, leveraging embeddings generated from associated text or visual content.
*   **Recommending Related Manuscript Pages:** When viewing a page of a digitized manuscript, the system suggests other pages with semantically similar content based on embeddings of the text or illuminations.

### 1.3 Terminology

This extension uses the following terms:

*   **embedded:** When an embedding vector is directly included within the JSON of the Embedding Annotation.
*   **referenced:** When an embedding vector is located at an external URI and its location is referenced within the Embedding Annotation.
*   **Embedding Vector:** A numerical representation of the semantic meaning or visual characteristics of a piece of content.
*   **Vector Encoding:** Specifies the serialization format of the data within the `vector` property of an embedded `EmbeddingVector`.
*   **Vector Data Type**: The underlying numerical data type of the individual elements in an embedding vector when encoded in a binary format (e.g., `float32`, `int16`). This term is typically described within the `model` property when using `base64` vector encoding.
*   **Vector Endianness**: The byte order (e.g., `little` or `big`) for multi-byte numerical data types within an embedding vector when encoded in a binary format. This term is typically described within the `model` property when using `base64` vector encoding for multi-byte data types.

The terms *array*, *JSON object*, *number*, *string*, and *boolean* in this document are to be interpreted as defined by the Javascript Object Notation (JSON) specification.

The key words *must*, *must not*, *required*, *shall*, *shall not*, *should*, *should not*, *recommended*, *may*, and *optional* in this document are to be interpreted as described in RFC 2119.

## 2. Embedding Vectors for IIIF Resources

### 2.1 Embedding Vectors

Embedding vectors are dense numerical representations that capture the semantic or visual characteristics of data. For IIIF resources, these vectors can represent the content of an entire Canvas, a specific region within a Canvas, or even the content of a specific Annotation.

## 3. Web Annotations for Embedding Vectors

The embedding vector information is stored in an Embedding Annotation using the Web Annotation Data Model. This section details the structure of an Embedding Annotation and its relationship to the IIIF Presentation API and Image API. It includes examples of the different properties and options.

### 3.1 Embedded vs. Referenced Annotations

Embedding Annotations can be included in a IIIF Presentation API response as part of an Annotation Page under the `annotations` property of a Canvas or other relevant resource. Alternatively, Embedding Annotations can exist independent of their targeted IIIF resource and can be retrieved by resolving their URI `id`.

When Embedding Annotations are to be included with a Canvas, implementers must have at least one Annotation Page in the `annotations` property of the targeted Canvas.

### 3.2 Embedding Annotation motivation

The `motivation` property declares the reason for creating the Embedding Annotation. The `motivation` property must be included on all Embedding Annotations and when included it must have the value `embedding`.

The `embedding` motivation is distinct from motivations like `describing` or `tagging`. While those motivations associate textual descriptions or keywords, the `embedding` motivation specifically links a numerical vector representation of the content's semantic meaning or visual characteristics. This allows for more nuanced semantic search and similarity analysis beyond simple keyword matching.

The linked data context provided with this document includes the formal JSON-LD 1.1 motivation extension, and the vocabulary provided with this document contains the formal vocabulary for the `embedding` motivation discussed above.

### 3.3 Embedding Annotation target

The `target` property describes the resource that the Embedding Annotation applies to. The value for `target` must either be a full IIIF resource, a single region within a IIIF resource represented as a Specific Resource, or a specific Annotation.

For Embedding Annotations embedded in the `annotations` property of a Canvas, the `target` must be the Canvas URI. When the desired target is a part of a Canvas represented by a Specific Resource, the `source` must be the Canvas URI. For Embedding Annotations that are external to the IIIF resource they target, implementers should embed the IIIF resource in the `target` property instead of referencing it.

Clients processing the embedding information for spatial resources such as Canvases or Image Services require the original height and width to correctly interpret region-based selectors. Implementers should add the `height` and `width` properties to their embedded spatial resources for consistency.

For non-spatial resources like Collections, Manifests, or text-based Annotations, height and width properties are not applicable and should be omitted.

Sometimes the targeted resource exists within a parent resource, such as a targeted Canvas that exists embedded in some Manifest. In these cases, it is important to maintain the link between them to access useful contextual information. Implementers may use the `partOf` property to reference the parent resource.

#### 3.3.1 Targeting the Full Resource

Example of an Embedding Annotation targeting an entire Canvas:

```json
{
  "id": "http://www.example.org/embedding-annotation1.json",
  "type": "Annotation",
  "motivation": "embedding",
  "target": "http://www.example.org/canvas1.json",
  "body": {
    "type": "EmbeddingVector",
    "vector": [0.12, -0.54, 0.87, 0.23, -0.91 ],
    "vectorEncoding": "json-array",
    "model": {
      "name": "multimodal-embedding-model",
      "version": "1.0",
      "dimensions": 5
    }
  }
}
```

When embedding the Canvas within the target:

```json
{
  "id": "http://www.example.org/embedding-annotation1.json",
  "type": "Annotation",
  "motivation": "embedding",
  "target": {
    "id": "http://www.example.org/canvas1.json",
    "type": "Canvas",
    "height": 2000,
    "width": 1000
    // ... other Canvas properties
  },
  "body": {
    "type": "EmbeddingVector",
    "vector": [0.12, -0.54, 0.87, 0.23, -0.91, ... ],
    "vectorEncoding": "json-array",
    "model": {
      "name": "multimodal-embedding-model",
      "version": "1.0",
      "dimensions": 5
    }
  }
}
```

#### 3.3.2 Targeting a Specific Region of the Resource

When a Specific Resource is used as a target, the `source` property supplies the resource and the `selector` property indicates the region of the resource that the annotation applies to. A IIIF Image API Selector or other types of Selectors can be used. Selectors used to target regions must conform to specifications like the IIIF Image API region parameters.

Example of an Embedding Annotation targeting a region of a Canvas:

```json
{
  "id": "http://www.example.org/region-embedding-annotation.json",
  "type": "Annotation",
  "motivation": "embedding",
  "target": {
    "type": "SpecificResource",
    "source": {
      "id": "http://www.example.org/canvas2.json",
      "type": "Canvas",
      "height": 2514,
      "width": 5965
    },
    "selector": {
      "type": "ImageApiSelector",
      "region": "100,100,200,200"
    }
  },
  "body": {
    "type": "EmbeddingVector",
    "vector": [0.45, 0.21, -0.73, 0.19, 0.88, ... ],
    "vectorEncoding": "json-array",
    "model": {
      "name": "multimodal-embedding-model",
      "version": "1.0",
      "dimensions": 5
    }
  }
}
```

#### 3.3.3 Targeting a Collection

Embedding vectors can be associated with Collections to enable semantic search across entire collections. Since Collections are non-spatial resources, they don't have height and width properties.

Example of an Embedding Annotation targeting a Collection:

```json
{
  "id": "http://www.example.org/collection-embedding.json",
  "type": "Annotation",
  "motivation": "embedding",
  "target": {
    "id": "http://www.example.org/collection/paintings.json",
    "type": "Collection"
  },
  "body": {
    "type": "EmbeddingVector",
    "vector": [0.42, 0.18, -0.63, 0.29, 0.75, ... ],
    "vectorEncoding": "json-array",
    "model": {
      "name": "multimodal-embedding-model",
      "version": "1.0",
      "dimensions": 5
    }
  }
}
```

#### 3.3.4 Targeting a Manifest

Embedding vectors can be associated with Manifests to enable semantic understanding of the entire resource. Like Collections, Manifests don't have height and width properties.

Example of an Embedding Annotation targeting a Manifest:

```json
{
  "id": "http://www.example.org/manifest-embedding.json",
  "type": "Annotation",
  "motivation": "embedding",
  "target": {
    "id": "http://www.example.org/manifest/book1.json",
    "type": "Manifest"
  },
  "body": {
    "type": "EmbeddingVector",
    "vector": [0.31, -0.58, 0.22, 0.79, -0.43, ... ],
    "vectorEncoding": "json-array",
    "model": {
      "name": "text-embedding-model",
      "version": "2.1",
      "dimensions": 5
    }
  }
}
```

#### 3.3.5 Targeting an Annotation

Embedding vectors can also be associated with the content of specific Annotations. This allows for semantic understanding of the Annotation's body.

Example of an Embedding Annotation targeting another Annotation:

```json
{
  "id": "http://www.example.org/annotation-embedding.json",
  "type": "Annotation",
  "motivation": "embedding",
  "target": {
    "id": "http://www.example.org/annotation/textual-description.json",
    "type": "Annotation"
  },
  "body": {
    "type": "EmbeddingVector",
    "vector": [0.32, -0.11, 0.52, -0.47, 0.21, ... ],
    "vectorEncoding": "json-array",
    "model": {
      "name": "text-embedding-model",
      "version": "2.1",
      "dimensions": 5
    }
  }
}
```

#### 3.3.6 Targeting a Range

Embedding vectors can be associated with Ranges to enable semantic understanding of structured parts of a resource. This is particularly useful for representing sections of books, chapters, movements in musical works, or other logical divisions.

Example of an Embedding Annotation targeting a Range:

```json
{
  "id": "http://www.example.org/range-embedding.json",
  "type": "Annotation",
  "motivation": "embedding",
  "target": {
    "id": "http://www.example.org/manifest/book1.json/range/chapter3",
    "type": "Range"
  },
  "body": {
    "type": "EmbeddingVector",
    "vector": [0.55, 0.12, -0.33, 0.67, -0.21, ... ],
    "vectorEncoding": "json-array",
    "model": {
      "name": "text-embedding-model",
      "version": "2.1",
      "dimensions": 5
    }
  }
}
```

### 3.4 Embedding Annotation body

The `body` of an Embedding Annotation **MUST** be a single JSON object. This object **MUST** have a `type` property with the value `EmbeddingVector`. The `EmbeddingVector` object is responsible for providing the embedding vector information and details about the `model` used. It **MUST** include:
1.  A `model` property as defined in Section 3.4.3.
2.  The actual vector data, provided either directly embedded using the `vector` property (see Section 3.4.1) or by an external reference using the `vectorReference` property (see Section 3.4.2).
An `EmbeddingVector` object **MUST NOT** contain both a `vector` property and a `vectorReference` property simultaneously.

The choice between embedding vectors directly within the Annotation body or referencing them externally depends on various factors, including the size of the vectors, their reusability, and architectural considerations.

#### 3.4.1 Embedded Vectors

Embedding vectors directly within the Annotation body includes the vector data directly in the JSON.

An example of the Embedding Annotation body with an **embedded vector**:

```json
{
  "body": {
    "type": "EmbeddingVector",
    "vector": [0.12, -0.54, 0.87, 0.23, -0.91, ... ],
    "vectorEncoding": "json-array",
    "model": {
      "name": "embedding-model-a",
      "version": "1.0",
      "dimensions": 3072,
      "type": "text",
      "normalization": true,
      "provider": "Provider A"
    }
  }
}
```

The `vector` property holds the embedded embedding vector data. The `vectorEncoding` property is **REQUIRED** if the `vector` property is present and specifies the serialization format of this data.

Currently, two encoding types are defined:

* **`"json-array"`**:
    * When `vectorEncoding` is `"json-array"`, the `vector` property **MUST** contain a JSON array where each element is a JSON number.
        *Example:* `"vector": [0.12, -0.54, 0.87, 0.23, -0.91]`
    * The numbers in the array represent the elements of the embedding vector in order.
    * The specific numeric precision of these numbers (e.g., as single-precision or double-precision floating-point values) is not defined by this encoding itself. Implementers *SHOULD* refer to metadata in the `model` property (e.g., `model.dimensions` for the expected length and potentially a model-specific description of its output precision) if this level of detail is critical.

* **`"base64"`**:
    * When `vectorEncoding` is `"base64"`, the `vector` property **MUST** contain a JSON string representing the Base64 encoded sequence of bytes of the vector.
    * The decoded byte sequence represents a packed, contiguous array of numerical values. The elements of the vector are packed in order.
    * To correctly interpret these bytes, the associated `model` property (see Section 3.4.3) **MUST** provide the following additional metadata fields:
        * `dataType` (e.g., `model.dataType`): A string specifying the numerical data type of each element in the vector. This field is **REQUIRED** for `base64` encoding. Recommended values include:
            * `"float32"`: IEEE 754 32-bit single-precision floating-point.
            * `"float64"`: IEEE 754 64-bit double-precision floating-point.
            * `"int8"`, `"uint8"`, `"int16"`, `"uint16"`, `"int32"`, `"uint32"`: Signed or unsigned integers of specified bit lengths.
            *(This list can be expanded, or you can reference an external standard like JavaScript TypedArray names).*
        * `endianness` (e.g., `model.endianness`): A string specifying the byte order for multi-byte data types. This field is **REQUIRED** for `base64` encoding if `dataType` refers to a multi-byte type (e.g., `float32`, `int16`) and **MUST** be omitted for single-byte types (e.g., `int8`, `uint8`). Recommended values are:
            * `"little"`: Little-Endian (least significant byte first).
            * `"big"`: Big-Endian (most significant byte first).
    * The total number of bytes in the decoded Base64 string **MUST** be consistent with the `model.dimensions` and the specified `model.dataType`. For example, if `model.dimensions` is 3 and `model.dataType` is `"float32"`, the decoded data must consist of 12 bytes (3 dimensions × 4 bytes per float32).
    * **Example of an Embedding Annotation body with a `base64` encoded vector:**
        ```json
        {
          "body": {
            "type": "EmbeddingVector",
            "vector": "AAAAAAAA8D8AAAAAAABAQC49z70=", // Example: [1.0, 2.0, 0.007] as float32 little-endian
            "vectorEncoding": "base64",
            "model": {
              "name": "my-binary-output-model",
              "version": "1.1",
              "dimensions": 3,
              "dataType": "float32",
              "endianness": "little",
              "type": "image"
            }
          }
        }
        ```

The `"base64"` encoding can be more compact than `"json-array"` for large vectors, especially when representing binary floating-point numbers, as it avoids the overhead of text representation for each number and JSON array delimiters. It can also simplify integration with systems that natively handle binary vector data. Clients consuming Base64 encoded vectors **MUST** use `model.dataType` and `model.endianness` (where applicable) to correctly reconstruct the numerical vector from the decoded byte stream.

#### 3.4.2 Externally Referenced Vectors

Referencing vectors externally links to the vector data via a URI.

The `vectorReference` property is defined by this document to hold the URI of an externally hosted embedding vector. The `format` property **MUST** be used to specify the [media type](https://en.wikipedia.org/wiki/Media_type) of the resource at the `vectorReference` URI. This informs the client how to retrieve and initially parse the external file. Common values include `"application/json"` (e.g., a file containing a JSON array of numbers), `"text/csv"`, or binary types such as `"application/octet-stream"`.

While the `format` property dictates how to read the file, the `model` property associated with the `EmbeddingVector` (see Section 3.4.3) provides essential metadata about the vector's intrinsic numerical characteristics. To correctly interpret the content of the externally referenced file as a numerical vector:

* The `model.dimensions` property **MUST** always be present to indicate the expected number of elements in the vector.
* The `model.dataType` property (e.g., `"float32"`, `"float64"`) **SHOULD** be included. It specifies the intended numerical data type of the vector elements as generated by the model.
    * For text-based formats (e.g., `format` is `"application/json"` or `"text/csv"`), `model.dataType` provides semantic clarity regarding the precision or type of the numbers represented in the text.
    * For binary formats (e.g., `format` is `"application/octet-stream"` or a specific binary vector type), `model.dataType` is **REQUIRED** to correctly parse the byte stream into numerical values.
* The `model.endianness` property (e.g., `"little"`, `"big"`) **MUST** be included if the `format` indicates a binary type *and* `model.dataType` refers to a multi-byte numeric type (e.g., `float32`, `int16`). This property is not applicable and **SHOULD** be omitted if the `format` is text-based or if `model.dataType` refers to a single-byte type (e.g., `int8`, `uint8`).

Clients **MUST** use the `format` property to determine how to initially parse the external file. Subsequently, they **MUST** use `model.dimensions`, `model.dataType`, and `model.endianness` (where applicable and provided) to correctly interpret and reconstruct the numerical vector from the file's content.

**Example 1: External vector in a JSON file**

This example references an external JSON file which is expected to contain a direct JSON array of numbers. The `model.dataType` informs about the nature of these numbers.

```json
{
  "body": {
    "type": "EmbeddingVector",
    "vectorReference": "http://www.example.org/embeddings/set_alpha/doc123_vector.json",
    "format": "application/json", // The file at this URI contains a JSON array, e.g., [0.123, -0.456, ...]
    "model": {
      "name": "universal-sentence-encoder-v4",
      "version": "4.0",
      "dimensions": 512,
      "dataType": "float32", // Indicates the intended precision/type of numbers in the JSON array
      "type": "text",
      "provider": "Provider Global"
    }
  }
}
```

**Example 2: External vector in a binary file**

This example references an external binary file containing a packed sequence of 32-bit floating-point numbers, little-endian.

```json
{
  "body": {
    "type": "EmbeddingVector",
    "vectorReference": "http://www.example.org/embeddings/image_features/img_abc.f32le", // ".f32le" is a conventional (not standard) extension indicating float32 little-endian
    "format": "application/octet-stream", // Or a future specific MIME type for binary vectors
    "model": {
      "name": "resnet50-feature-vector",
      "version": "2.1",
      "dimensions": 2048,
      "dataType": "float32",     // REQUIRED for interpreting the binary stream
      "endianness": "little",   // REQUIRED for interpreting the binary stream for float32
      "type": "image",
      "normalization": true,
      "provider": "Provider Vision"
    }
  }
}
```

#### 3.4.3 The `model` Property

The `model` property is a required JSON object that provides information about the embedding model used to generate the vector. Without this information, clients cannot properly interpret or compare the embedding vectors. The `model` property must include:

*   **`name`:** The name or identifier of the embedding model (e.g., "embedding-model-a", "embedding-model-b").
*   **`version`:** The version of the embedding model.

The `model` property **may also include** additional properties to further specify how the embeddings were generated:
* **`dimensions`**: The number of dimensions in the embedding vector (e.g., 768, 1024, 1536, 3072). This property **MUST** be present if the `vectorEncoding` is `"base64"` (see Section 3.4.1 for requirements) or if the `vectorReference` property is used (see Section 3.4.2). For `"json-array"` encoding, while the vector length can be inferred from the array, including `dimensions` in the `model` is **RECOMMENDED** for explicit clarity and consistency.
*   **`dataType`**: A string specifying the numerical data type of each element in the vector (e.g., `"float32"`, `"float64"`). See Section 3.4.1 for recommended values. This property is **REQUIRED** when `vectorEncoding` is `"base64"`.
*   **`endianness`**: A string specifying the byte order for multi-byte data types (e.g., `"little"`, `"big"`). See Section 3.4.1 for details. This property is **REQUIRED** when `vectorEncoding` is `"base64"` and `dataType` is a multi-byte type.
*   **`type`**: The type of embedding model (e.g., "text", "multimodal", "image").
*   **`truncation`:** How text longer than the model's maximum token limit was handled (e.g., "none", "start", "end").
*   **`normalization`:** Whether the vectors were normalized (boolean).
*   **`provider`:** Organization or service that provided the embedding model (e.g., "Provider A", "Provider B").
*   **`maxTokens`:** The maximum number of tokens used in generating this embedding.

Example model property for a multimodal embedding:

```json
"model": {
  "name": "embedding-model-b",
  "version": "1.0",
  "dimensions": 1536,
  "type": "multimodal",
  "normalization": true,
  "provider": "Provider B"
}
```

## 4. Linked Data Context

The URI of this extension's linked data context would be `http://iiif.io/api/extension/embeddings/1/context.json`
The URI of the IIIF Presentation API linked data context is `http://iiif.io/api/presentation/3/context.json`

The linked data context of this extension must be included before the IIIF Presentation API linked data context on the top-level object.

**Example `context.json` for the Embeddings Extension:**

```json
{
  "@context": {
    "dcterms": "http://purl.org/dc/terms/",
    "motivation": { "@id": "http://www.w3.org/ns/oa#motivation", "@type": "@id" },
    "oa": "http://www.w3.org/ns/oa#",
    "exemb": "http://iiif.io/api/extension/embeddings/vocab/",
    "EmbeddingVector": "exemb:EmbeddingVector",
    "vector": "exemb:vector",
    "vectorReference": { "@id": "exemb:vectorReference", "@type": "@id" },
    "vectorEncoding": "exemb:vectorEncoding",
    "model": "exemb:model",
    "name": "dcterms:name",
    "version": "dcterms:version",
    "dimensions": "exemb:dimensions",
    "dataType": "exemb:dataType",
    "endianness": "exemb:endianness",
    "type": "exemb:type",
    "normalization": "exemb:normalization",
    "provider": "exemb:provider",
    "maxTokens": "exemb:maxTokens",
    "truncation": "exemb:truncation",
    "embedding": { "@id": "exemb:embedding", "@type": "@id" }
  }
}
```

**Vocabulary URI:** `http://iiif.io/api/extension/embeddings/vocab/`

This vocabulary defines the following terms:

*   **`dataType`**: A property specifying the numerical data type of each element in an embedding vector, particularly for binary encoded vectors.
*   **`dimensions`**: A property specifying the number of dimensions in the embedding vector.
*   **`embedding`**: The vocabulary term for the `embedding` motivation.
*   **`EmbeddingVector`**:  A type indicating that the body contains semantic vector information.
*   **`endianness`**: A property specifying the byte order for multi-byte numerical data types in an embedding vector, particularly for binary encoded vectors.
*   **`maxTokens`**: A property indicating the maximum number of tokens used in generating the embedding.
*   **`model`**: A property that provides metadata about the embedding model.
*   **`name`**: A property defining the name of the embedding model. This term reuses dcterms:name (http://purl.org/dc/terms/name).
*   **`normalization`**: A property indicating whether the vectors were normalized.
*   **`provider`**: A property indicating the organization or service that provided the embedding model.
*   **`truncation`**: A property indicating how text longer than the model's maximum token limit was handled.
*   **`type`**: A property indicating the type of embedding model (text, multimodal, etc.).
*   **`vector`**: A property holding the embedded vector data.
*   **`vectorEncoding`**: A property indicating the encoding of the embedded vector.
*   **`vectorReference`**: A property holding the URI of an externally referenced vector.
*   **`version`**: A property defining the version of the embedding model. This term reuses dcterms:version (http://purl.org/dc/terms/version).
