# Open Container Initiative

## Artifacts Specification

This Artifacts Spec provides a reference for artifact authors and registry implementors to support new artifact types with pre 1.0 implementations of the [distribution-spec][distribution-spec]. More particularly this repository (spec) has been tasked by the OCI TOB to serve 3 primary goals:

1. **Artifact Authors** - guidance for authoring new artifact types. Including a clearing house for well known artifact types.
1. **Registry Operators and Vendors** - guidance for how operators and vendors can support new artifact types, including how they can opt-in or out of well known artifact types. Registry operators that already implement `mediaType` filtering will not have to change. The artifact repo will provide context on how new `mediaTypes` can be used, and how `mediaTypes` can be associated with a type of artifact.
1. [**Clearing House for Well-known Artifacts**][def-well-known-types] - artifact authors can submit their artifact definitions, providing registry operators a list by which they can easily support.

[Artifacts v1.0][artifacts] presumes a refactoring of the [OCI Image-spec 1.0][image-spec] and the pending [OCI Distribution-spec 1.0][distribution-spec] to refactor [manifest][image-manifest] and [index][image-index] to seperatly versioned specs.

**Reference**:

- [Need a Working Group for Updating the distribution-spec #9](https://github.com/opencontainers/artifacts/issues/9)
- [Need a Working Group for Updating the image-spec #10](https://github.com/opencontainers/artifacts/issues/10)

**v1.0 of the image-spec** defines index and manifest.

![image-spec](./media/image-spec-objects.png)

**Artifacts 1.0** visualzies a refactoring of index and manifest.

![refactored artifacts spec](./media/artifacts-spec-objects.png)

## Table of Contents

- [Scope](#scope)
  - [Future](#future-scope)
- [Defining OCI Artifact Types](#defining-oci-artifact-types)
- [Defining a Unique Artifact Type](#defining-a-unique-artifact-type)
- [Defining Supported Layer Types](#defining-supported-layer-types)
- [Optional: Defining Config Schema](#optional-defining-config-schema)
- [Optional: Artifact Publisher Manifest](#optional-defining-artifact-publisher-manifests)
  - [Artifact Publisher Manifest Example](#artifact-publisher-manifest-type-example) as a [Well Known Type][def-well-known-types]

## Scope

[Artifacts v1.0][artifacts] is dependent on [OCI manifest][image-manifest] 1.0 to represent individual artifacts including [OCI Image][image-spec], [Helm][helm] and [Singularity][singularity].

### Future Scope

Future versions of artifacts will support new artifacts that represent collections of artifacts using [OCI Index][image-index].

To support artifacts that represent a collection of other artifacts, a means to identify an index as a type of artifact will be required.

## Defining OCI Artifact Types

As registries become content addressable distribution points, tools that pull artifacts must know if they can operate on the artifact. Artifact types are equivalent to file extensions. When users open files, the host operating system typically launches the appropriate program. When users open a file, from within a program, the open dialog filters to the supported types. When search or security software scan the contents of a storage solution, the software must to know how to process the different types of content. When users view the contents of a storage solution, they see the textual and visual indications of the type. OCI Artifacts provides these core capabilities to [OCI distribution spec][distribution-spec] based registries.

Authoring an OCI Artifacts involves the following steps:

- Define a unique type
- Define the format for other tools to operate upon the type
- Define human elements, such as an icon and localized string to be displayed to users
- Optionally, publish as a [well-known type][def-well-known-types] for registries to consume

Defining a unique type, string and logo; artifacts can be represented as the following:

|Icon|Artifact|`config.mediaType`|
|-|-|-|
|<img src="https://github.com/opencontainers/artwork/blob/master/oci/icon/color/oci-icon-color.png?raw=true" width=30x>|[OCI Image][image-spec]|`application/vnd.oci.image.config.v1+json`|
|<img src="https://github.com/helm/helm-www/blob/master/themes/helm/static/img/apple-touch-icon.png?raw=true" width=30x>|[Helm Chart](https://helm.sh)|`application/vnd.cncf.helm.chart.config.v1+json`|
|<img src="https://github.com/sylabs/singularity/blob/master/docs/logos/singularity_v3.png?raw=true" width=30x>|[Singularity][singularity], by [Sylabs][sylabs]|`application/vnd.sylabs.sif.config.v1+json`|

## Defining a Unique Artifact Type

Defining the unique type involves uniqueness for computer processing, and uniqueness for humans.

For computer processing, artifacts are defined by setting `manifest.config.mediaType` to a globally unique value.

> **Note:** The `config.mediaType` of `application/vnd.oci.image.config.v1+json` is reserved for artifacts intended to be instanced by [docker][docker], [containerd][containerd] and other [OCI Image][image-spec] toolchains.  

Each artifact MUST have its own unique type.

 The following `mediaType` format is used to differentiate the type of artifact:

`application/vnd.`[org|company]`.`[objectType]`.`[optionalSubType]`.config.`[version]`+json`

- **`org|company`** - represents an open source foundation (`oci`, `cncf`) or a company (`microsoft`, `ibm`).
- **`objectType`** - a value representing the short name of the type. The combination of the org|company and object type should provide human identification and uniqueness for computers indexing.
- **`optionalSubType`** - provides additional extensibility of an `objectType`
- **`version`** - provides artifact authors the ability to revision their schemas and formatting, enabling tools to identify which format they will process.

## Defining Supported Layer Types

Artifacts are intended to have content. The content of an artifact is represented through a collection of blobs as provided by the [distribution spec][distribution-spec]. How the blobs are re-constituted, and whether the blobs are ordinal layers is a decision of the artifact author.

As an example, [OCI Images][image-layer] are represented through an ordinal collection of tar archives. Each blob represents a layer. Each layer overlays the previous layer. These blobs may be further compressed.

**Note**: *For the purposes of this spec, the terms **layer** and **blob** are interchangable.*

Other artifacts may be represented as a single file, such as config artifact representing a deployment specification. Other artifacts may include a config blob, and a collection of binaries compressed as another blob. By separating the blob, the artifact author can benefit from layer de-duplication and concurrent downloading of each blob.

### Layer File Format

Layers are persisted as blobs within registries. The blobs can be a single file or a collection of files (`.tar`). The blob format is up to the artifact author. They may choose to persist individual files with standard or custom formats, such as `.config`, `.json`, `.bin`, `.sif`, or store a collection of files as `.tar` archive. Optionally, the author MAY choose to compress the blob. The extension of the `layer.mediaType` MUST relfect the format of the blob.

### Layer Versioning

Layers MUST define a version to future proof new enhancements that may evolve. How the artifact tooling processes forward and backward compatibility is up to the artifact and tooling authors.

### Defining Layer Types

Artifact **layers types** are not expected, nor required to be shared across different **artifact types**. A layer type that represents an OCI Image is not expected to be shared with a Helm Chart. However, two helm charts that have the same content MAY be de-duped within the registry. The **layer type** is the defintion of the contents. To differentiate layers between artifact types, artifact authors SHOULD define unique artifact layer types.

Artifact layer types follow the same convention as manifests, utilizing the `layer.mediaType` with the following format:  
`application/vnd.`[org|company]`.`[layerType]`.`[layerSubType]`.layer.`[version]`.`[fileFormat]`+`[OPTIONAL compressionFormat]

**Note:** optional compression of the type is represented as: `+`[compressionFormat]

The `mediaType` extension MUST reflect the content format of the blob.

### Example Layer Types

|Artifact Type|mediaType|
|-|-|
|Non-compressed OCI Image Layer|`application/vnd.oci.image.layer.v1.tar`|
|[gzip] Compressed OCI Image Layer|`application/vnd.oci.image.layer.v1.tar+gzip`|
|[gzip] Compressed Docker Image Layer|`application/vnd.docker.image.rootfs.diff.tar+gzip`|
|Non-compressed Helm Chart Layer|`application/vnd.cncf.helm.chart.layer.v1.tar`|
|[gzip] Compressed Helm Chart Layer|`application/vnd.cncf.helm.chart.layer.v1.tar+gzip`|
|Singularity SIF Layer|`application/vnd.sylabs.sif.layer.v1.sif`|
|Sample Text File Layer|`application/vnd.sample.text.layer.v1.txt`|
|[gzip] Compressed Sample Text File Layer|`application/vnd.sample.text.layer.v1.txt+gzip`|
|[zstd] Compressed Sample Text File Layer|`application/vnd.sample.text.layer.v1.txt+zstd`|

## Optional: Defining Config Schema

While the value of `manifest.config.mediaType` is used to determine the artifact type, the persistance of a `config.json` file is OPTIONAL. Artifacts can push a null reference for `config.json` persistance.

When defining an artifact type, the persistance of the artifact may be broken up into content and configuration. Configuration can be stored as a blob, or it can be stored and referenced by the `manifest.config`.

Some benefits of using `manifest.config` include:

- Tooling can pull configuration prior to any blobs. Using the config to determine how and where the blob should be instanced, an artifact tool might send blob request to another compute instance, such as OCI Image layers being sent to a Windows or Linux Host.
- Registries may opt-into parsing the configuration if it provides meaningful top-level information. [OCI Image Config][image-spec-config] stores `OS`, `Architecture` and `Platform` information that some registry operators may wish to display. The config is easy to pull & parse, as opposed to getting a layer url to pull, possibly decompress and parse.

Distribution instances MAY:

- Parse and process the contents of  `manifest.config`, based on the provided schema of `manifest.config.mediaType`, offering additional information or actions.
- Ignore the contents and validation of the config.json file.

## Optional: Defining Artifact Publisher Manifests

Artifact authors may wish to publish their types as [well known types][def-well-known-types] for registry operators to consume. By publishing an `artifactManifest.json`, registry operators can parse information for validation, registry display and search results.

An `artifactType.json` file is defined using the [following schema](artifactTypes/artifactTypeSchema.0.1.json).

- **`mediaType`** *string*  
  This REQUIRED property uniquely identifies the artifact for computer consumption. It may be owned by an org or a company and MUST be globally unique and versioned.  
  The format of `mediaType` MUST use the following format:  
  `application/vnd.`[org|company]`.`[objectType]`.`[optionalSubType]`.config.`[version]`+json`
- **`title`** *string-string map*  
  This REQUIRED property must have at least one value, representing the name of the type displayed for human consumption. The title may be displayed in a repository listing, or registry tooling.  
  Title is a collection of localized strings, indexed with [ISO Language Codes][iso-lang-codes].
  - **`locale`** *string*  
    2-2 language codes representing the country and locale. [ISO Language Codes][iso-lang-codes]
  - **`title`** *string*  
Localized title. The max length MUST not exceed 30 characters and MUST not encode formatting characters.
- **`description`** *string-string map*  
    This REQUIRED property must have at least one value, providing a short description of the type for human consumption. The description may be displayed in repository listings or registry tooling.  
    Description is a collection of localized strings, indexed with [ISO Language Codes][iso-lang-codes].  
  - **`locale`** *string*  
    2-2 language codes representing the country and locale. [ISO Language Codes][iso-lang-codes]
  - **`description`** *string*  
Localized description. The max length MUST not exceed 255 characters and MUST not encode formatting characters.
- **`moreInfoUrl`** *url*  
This OPTIONAL property provides additional details, intended for consumers of the type. This is most often a marketing & awareness overview page.
- **`specUrl`** *url*  
This OPTIONAL property references a spec, providing additional details on the type.
- **`tools`** *string-string map*  
  This OPTIONAL property provides a collection of tools that may be used with artifact type. The property is intended for end users to find more info on how to find and install related tools. Registry operators MAY provide links to the tools in their repository listing.
  - **`url`** *url*  
    This REQUIRED property links to a page where users can download the tool. The URL MAY be a direct link to a download URL, or a link to documentation for how to download the tool.
  - **`title`** *string-string map*  
    This OPTIONAL property representes the name of the tool, displayed for human consumption. The title may be displayed in a repository listing, or registry tooling.  
    Title is a collection of localized strings, indexed with [ISO Language Codes][iso-lang-codes].
    - **`locale`** *string*  
      2-2 language codes representing the country and locale. [ISO Language Codes][iso-lang-codes]
    - **`title`** *string*  
Localized title. The max length MUST not exceed 30 characters and MUST not encode formatting characters.
- **`configSchemaReferenceUrl`** *url*  
This OPTIONAL property provides a schema reference for the artifact config object. The schema is provided for registry operators and tools to optionally validate and process information within the config. A registry operator MAY wish to present information, such as the OCI image architecture type. Each versioned artifact type would have a unique version, possibly referencing a unique schema version. To version the schema, the artifactType MUST also be versioned.
- **`layerMediaTypes`** string-string map  
  This REQUIRED property must have at least one value, representing one or more layer `mediaTypes` used by the artifact.  
  Layer mediaTypes SHOULD be unique to the specific artifact.  
  Layer mediaTypes are NOT REQUIRED to be unique across different artifact types when artifacts intend to share layers across different artifact tooling.  
  Registry operators MAY choose to validate layers associated with a specific artifact type. Providing the supported layers enables registry operators to know the supported `mediaTypes`.

  `layerMediaTypes` use the following format:  
  `application/vnd.`[org|company]`.`[objectType]`.`[optionalSubType]`.layer.`[version]`+`[fileFormat].
  - **`mediaType`** *string*  
    This REQUIRED property represents a valid layer `mediaTypes` for the artifact.

### Artifact Publisher Manifest Type Example

The following is an example of an unknown artifact type.

```json
{
  "mediaType": "application/vnd.unknown.config.v1+json",
  "spec": "https://github.com/opencontainers/artifacts",
  "title": {
    "locale": "en-US",
    "title": "An unknown type"
  },
  "description": {
    "locale": "en-US",
    "description": "An undefined type - USE ONLY FOR DEVELOPMENT"
  },
  "url": "https://github.com/opencontainers/artifacts",
  "tools":[
    {
      "title": {
        "locale": "en-US",
        "title": "ORAS"
      },
      "url": "https://github.com/deislabs/oras"
    }
  ],
  "configSchemaReference": "",
  "layerMediaTypes": [
    "application/vnd.oci.unknown.layer.v1.bin",
    "application/vnd.oci.unknown.layer.v1.json",
    "application/vnd.oci.unknown.layer.v1.tar",
    "application/vnd.oci.unknown.layer.v1.txt",
    "application/vnd.oci.unknown.layer.v1.yaml"
  ]
}
```

[annotations]:          https://github.com/opencontainers/image-spec/blob/master/annotations.md
[artifacts]:            README.md
[containerd]:           https://containerd.io/
[def-registry]:         definitions-terms.md#registry
[def-well-known-types]: definitions-terms.md#well-known-types
[def-yass]:             definitions-terms.md#yass
[distribution-spec]:    https://github.com/opencontainers/distribution-spec/
[docker]:               https://github.com/moby/moby
[gzip]:                 https://tools.ietf.org/html/rfc1952
[helm]:                 https://github.com/helm/helm
[image-manifest]:       https://github.com/opencontainers/image-spec/blob/master/manifest.md
[image-index]:          https://github.com/opencontainers/image-spec/blob/master/image-index.md
[image-layer]:          https://github.com/opencontainers/image-spec/blob/master/layer.md
[image-spec]:           https://github.com/opencontainers/image-spec/
[image-spec-config]:    https://github.com/opencontainers/image-spec/blob/master/config.md
[distribution]:         https://github.com/docker/distribution
[singularity]:          https://github.com/sylabs/singularity
[sylabs]:               https://sylabs.io/
[zstd]:                 https://tools.ietf.org/html/rfc8478