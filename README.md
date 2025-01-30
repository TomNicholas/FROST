# FROST (Federated Registry Of Scientific Things)

FROST is a decentralized global data catalog protocol for sharing all scientific data.

> [!WARNING]  
> This doesn't actually exist yet, this repo is just for brainstorming ideas.

### Motivation

See the [blog post](https://hackmd.io/@TomNicholas/H1KzoYrPJe)

### Concept:

- Decentralized network protocol for disseminating updates to version-controlled datasets

### Idea:

- Global catalog of version-controlled Icechunk/Iceberg etc. datasets
- Shared as a decentralized / federated network spanning different organisations, so that no one organisation controls the catalog
- Publish updates to catalog entries, which then propagate through the network between organisations
- Allows datasets to subscribe to changes in upstream datasets
- Doesn’t store or transfer any data, just sends updates about which parts of it have changed

### Inspiration:

- Publisher-Subscriber Model
    - WebSub
    - RSS
- Shared networks
    - NNTP
- Decentralized Social Media protocols
    - ActivityPub
    - AT Protocol
- Data catalog protocols
    - [RDF](https://www.techtarget.com/searchapparchitecture/definition/Resource-Description-Framework-RDF#:~:text=The%20Resource%20Description%20Framework%20)
    - https://projects.eclipse.org/proposals/eclipse-dataspace-protocol
    - https://www.w3.org/TR/vocab-dcat-3/
- Open data catalogs
    - STAC
    - Intake
- Data catalog products
    - https://github.com/amundsen-io/amundsen
    - DataHub
    - https://atlan.com/what-is-a-data-catalog/
- Linked data concepts
    - https://json-ld.org/
    - https://en.wikipedia.org/wiki/Linked_data#Linked_open_data
    - https://en.wikipedia.org/wiki/Knowledge_commons

### Goals:

- Version-controlled
- Subscribable
- Searchable
- Decentralized

### Non-goals:

- Data storage
- Implementing the version-controlled data model
- Domain-specific features (e.g. recording spatio-temporal coverage of a dataset)
- Social network features beyond subscribing (e.g. commenting, liking)
- Doing any loading/computation/anything else once update is received

### Technical requirements:

- Entries
    - Uniquely-identifiable
    - Owned by some user/org
    - Versioned
    - Point to remote data
    - Data model-agnostic
        - Support Icechunk initially but extensible to Iceberg + other cloud-native version-controlled data models
    - Arbitrary metadata
        - e.g. STAC-compatible
    - Public or private visibility
- Updates
    - Uniquely-identifiable
    - Don’t send actual data
    - Describe which part of the data has changed (e.g. which chunks)
      - Q: Is this necessary? Perhaps all that's needed is a notification that _something_ changed?
- Actions
    - Publishing updates
    - Subscribing to updates
- Authentication
    - Authentication required to create or update entries
    - Public or private entries
    - Private entries and updates to private entries are invisible (outside of the host instance)
        - Q: Is any of this actually required? Or could host instances just be forced to implement this feature themselves internally if they want it?
- Latency
    - Updates should reach subscribers within minutes, ideally within seconds
- Eventual consistency
    - Updates should be guaranteed to eventually reach subscribers
- Scalability
    - Hosting
    - Network
        - Arbitrary size
- Right to Exit
    - Federated or Decentralized
    - Public entries and updates are all always available
- Features
    - Citable
        - Each entry has a DOI
    - Licenses
- Portable spec
    - Follow RFC2119
    - Language-agnostic
        - Do everything in JSON or similar

### Protocol Implementation:

- Piggyback off an existing protocol
    - ATproto
    - ActivityPub
    - Q's:
      - Which of these have the right amount of "federation"?
      - Do these have the ability to send arbitrary JSON payloads (to encode information about our updates)
- Write a dedicated protocol

### FAQ’s:

**Q: Why does it need to be unified across fields of science?**

A: Any attempt to split up the catalog by fields of science will inevitably divide some community’s interdisciplinary field in two. That's fine at the labelling level, but not cool to force that community to bridge two separate networks to receive all updates they care about. Your data probably isn’t that special anyway, you almost certainly could fit it into this framework.

**Q: Can’t we just use STAC?**

A: No, it’s not general enough (lots of scientific data that isn’t a Spatio-Temporal view of the Earth). See also the section in the motivation blog post.

**Q: Could I catalog X type of data with this?**

A: Only if that data can be represented via some open cloud-native version-controlled common model, e.g:
  - Icechunk for multidimensional array data
  - Iceberg for for tabular data
  - Perhaps LakeFS for unstructured blobs?

**Q: Why so tied to Icechunk/Iceberg?**

A: The version-controlled part is crucial, otherwise updates are not well-defined. The storage formats also need to be scalable and S3-accessible, which Icechunk/Iceberg are.

### License

[Apache 2.0](https://github.com/TomNicholas/FROST/blob/main/LICENSE)
