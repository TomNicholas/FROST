# FROST (Federated Registry Of Scientific Things)

FROST is a decentralized global data catalog protocol for sharing all scientific data.

> [!WARNING]  
> This doesn't actually exist yet, this repo is just for brainstorming ideas. Please contribute!

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
    - https://www.proto-okn.net/

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
- Network invariants
  - Must always be a simple graph
    - Or do we allow cycles?
  - No node is downstream of a now-deleted node

### Protocol Implementation:

- Piggyback off an existing protocol
    - ATproto
    - ActivityPub
    - Q's:
      - Which of these have the right amount of "federation"?
      - Do these have the ability to send arbitrary JSON payloads (to encode information about our updates)
- Write a dedicated protocol

### MVPs

1. **Hello World**

A network with a single dataset node. It is periodically updated, and publishes the fact it has been updated.

2. **Related data**

A network with two dataset nodes, one which refers to the other as being related in some way, so the graph has a single edge.

3. **Leader-Follower**

A network with two dataset nodes, one downstream which refers to the other upstream, stating that the downstream one has been derived from the upstream one in some specific programatic way, which is retriggered upon each update to the upstream dataset. The graph has a single edge.

4. **Cross-Org Catalog**

A network with two nodes, belonging to different organisations, that are not connected. We check that both nodes can be listed by both orgs.

5. **Cross-Org Follower**

A network with two nodes, belonging to different organisations, one downstream which refers to the other upstream, stating that the downstream one has been derived from the upstream one in some specific programatic way, which is retriggered upon each update to the upstream dataset. The graph has a single edge. We check that both nodes can be listed by both orgs.

### FAQ’s:

**Q: Why does it need to be unified across fields of science?**

A: Any attempt to split up the catalog by fields of science will inevitably divide some community’s interdisciplinary field in two. That's fine at the labelling level, but not cool to force that community to bridge two separate networks to receive all updates they care about. Your data probably isn’t that special anyway, you almost certainly could fit it into this framework.

**Q: Can’t we just use STAC?**

A: No, it’s not general enough (lots of scientific data that isn’t a Spatio-Temporal view of the Earth). See also the section in the motivation blog post.

**Q: Could I catalog <X> type of data with this?**

A: The set of allowed data models should be extensible, but restricted to any which have the following properties:
  - version-controlled at rest in object storage, with a uniquely identifiable address/hash for each commit (i.e. icechunk, git itself)
  - some idea of a diff, potentially one that's small enough to be sent over the network (e.g. git diff, icechunk's ChangeSet)
  - bytes can be pulled out via http range requests to a storage URL (so it can be accessed via an S3-compatible API)
  - be able to store very large amounts of data
  - store arbitrary metadata, e.g. a JSON with no pre-specified schema (zarr/icechunk has this)

Important examples which should already meet these criteria are:
  - Icechunk (multidimensional arrays)
  - Iceberg (tabular)
  - Perhaps LakeFS (unstructured blobs)?

**Q: I want the catalog layer to have a field for <my domain-specific metadata tag>**

No, bad. It's crucial that the catalog remain domain-agnostic. Adding domain-specific choices in catalog schema is one of the main reasons why so many existing projects in this space don't generalize. 

Instead this as a problem to be solved at the level of metadata standards. With the data catalog able to attach arbitrary metadata (e.g. JSON), the field of microscopists can work out amongst themselves some convention for the standard schema of their metadata and what that means to microscopists, whilst climate and weather people can make sure their data follows the CF conventions and so on. This approach is the only one compatible with what a standard _is_ - a community-agreed schema that is extremely useful when followed but you're not forced to follow it.

### License

[Apache 2.0](https://github.com/TomNicholas/FROST/blob/main/LICENSE)
