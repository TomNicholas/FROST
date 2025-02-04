# FROST (Federated Registry Of Scientific Things)

FROST is a decentralized subscribable data catalog protocol for sharing all scientific data globally.

![updating](https://github.com/user-attachments/assets/2e864f8f-7a29-4ea7-88a7-6b2365071869)

(See [the FAQ](https://github.com/TomNicholas/FROST?tab=readme-ov-file#faqs) for explanation of the GIF)

> [!WARNING]  
> This doesn't actually exist yet, this repo is just for brainstorming ideas. Please contribute!

## Motivation

**Context:**
The best way to store and distribute access to big scientific datasets is via [ARCO data](https://medium.com/pangeo/step-by-step-guide-to-building-a-big-data-portal-e262af1c2977) in S3-compatible storage. We now have scalable cloud-optimised formats that are version-controlled at rest in object storage (particularly [Icechunk](https://icechunk.io/) for arrays and [Iceberg](https://iceberg.apache.org/) for tables). This is _huge_, as even dynamically-updated datasets can now be distributed via raw S3, with no other server needed. All the data providers who are paying attention are about to put their data in these formats, and then they will all immediately try to advertise the S3 URLs to the world via ad-hoc data catalogs.

**Problem: Everyone's catalogs are disconnected from everyone else's**. 

This means:
- No cross-org discoverability (e.g. NASA catalog users won't see NOAA datasets or vice versa).
- No cross-org tracking of updates (e.g. NOAA datasets derived directly from NASA datasets won't automatically know if the NASA datasets have been updated upstream).
- Risk of "catalog wars" where platform services compete to make more and more comprehensive "meta-catalogs" which merely track (outdated) links to other orgs' data in S3.
- Risk that if one platform does win everyone might feel locked in to it via the social network effect.

**Solution: Federated catalog protocol with cross-org publish-subcribe model.** 

- Cross-org discoverability enabled via displaying the contents of the dataset entries being broadcast,
- Cross-org tracking of updates to datasets enabled the same way,
- No need to compete to make a better catalog, as anyone can easily consume and display the entire global catalog, including updates,
- Federated trust model allows proliferation of high-quality centralized services, whilst also guarding against platform lock-in.

**How do we build it?:** Not sure exactly, but the problem is analogous to creating Federated alternatives to centralized social media (i.e. Bluesky/Mastodon vs Twitter). Perhaps we can piggyback off of Bluesky's ATproto or Mastodon's ActivityPub?

See also the [motivating blog post](https://hackmd.io/@TomNicholas/H1KzoYrPJe)

## Concept:

- Decentralized network protocol for disseminating updates to version-controlled datasets

## Idea:

- Global catalog of version-controlled Icechunk/Iceberg etc. datasets
- Shared as a decentralized / federated network spanning different organisations, so that no one organisation controls the catalog
- Publish updates to catalog entries, which then propagate through the network between organisations
- Allows datasets to subscribe to changes in upstream datasets
- Doesn’t store or transfer any data, just sends updates about which parts of it have changed

## Inspiration:

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

## Goals:

- Version-controlled
- Subscribable
- Searchable
- Decentralized (or at least Federated)

## Non-goals:

- Data storage
- Implementing the version-controlled data model
- Domain-specific features (e.g. recording spatio-temporal coverage of a dataset)
- Social network features beyond subscribing (e.g. commenting, liking)
- Doing any loading/computation/anything else once update is received

## Technical requirements:

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

## Protocol Implementation:

- Piggyback off an existing protocol
    - ATproto
    - ActivityPub
    - Q's:
      - Which of these have the right amount of "federation"?
      - Do these have the ability to send arbitrary JSON payloads (to encode information about our updates)
- Write a dedicated protocol

## MVPs

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

## FAQ’s:

**Q: That GIF is pretty, but what does it mean?**

The GIF is intended to show notifications of dataset updates propagating through a federated network. 

Each node is a version-controlled dataset sitting in S3, in either Icechunk or Iceberg format. The datasets are spread across 3 organisations: NASA, NOAA, and a startup (rocketship). Although each dataset sits in the owning organisation's object storage, the locations, versions, and dependencies of each are shared publicly via the FROST protocol. They thus form a cross-org (federated) network, the FROST network.

A source of new data (the satellite) causes the NASA dataset to be updated. A notification of this update is broadcast to it's dependent datasets. A re-computation of these dependents is triggered, and updated versions of each written out.

**Q: Why does it need to be unified across fields of science?**

A: Any attempt to split up the catalog by fields of science will inevitably divide some community’s interdisciplinary field in two. That's fine at the labelling level, but not cool to force that community to bridge two separate networks to receive all updates they care about. Your data probably isn’t that special anyway, you almost certainly could fit it into this framework.

**Q: Can’t we just use STAC?**

A: No, it’s not general enough (lots of scientific data that isn’t a Spatio-Temporal view of the Earth). See also the section in the motivation blog post.

**Q: Could I catalog `<some data type>` with this?**

A: The set of allowed data models should be extensible, but restricted to those which have the following properties:
  - version-controlled at rest in object storage, with a uniquely identifiable address/hash for each commit (i.e. icechunk, git itself)
  - some idea of a diff, potentially one that's small enough to be sent over the network (e.g. git diff, icechunk's `ChangeSet`)
  - allows bytes can be pulled out via http range requests to a storage URL (so it can be accessed via an S3-compatible API)
  - be able to store very large amounts of data (this and the previous requirement amount to it being "cloud-optimised")
  - store arbitrary metadata, e.g. a JSON with no pre-specified schema (zarr/icechunk has this)

Important examples which should already meet these criteria are:
  - Icechunk (multidimensional arrays)
  - Iceberg (tabular)

**Q: Can the catalog layer have a field for `<my domain-specific metadata tag>`?**

A: No, bad. It's crucial that the catalog remain domain-agnostic. Adding domain-specific choices in catalog schema is one of the main reasons why so many existing projects in this space don't generalize.

Instead this as a problem to be solved at the level of metadata standards. With the data catalog able to attach arbitrary metadata (e.g. JSON), the field of microscopists can work out amongst themselves some convention for the standard schema of their metadata and what that means to microscopists, whilst climate and weather people can make sure their data follows the CF conventions and so on. This approach is the only one compatible with what a standard _is_ - a community-agreed schema that is extremely useful when followed but you're not forced to follow it.

**Q: But shouldn't we enforce that the data at least has `<requirement>`?**

A: No. Basically nothing other than the bare minimum for the system to work should be enforced. As soon as you enforce anything it raises the barrier to entry, reducing adoption.
Your enforcement will also inevitably bake in some assumptions that seem reasonable in your field but aren’t meetable in general, so you end up making it less generalizable.
Note that GitHub enforces nothing, not even having a license or readme (though it does very strongly suggest them). It doesn’t try to force you to use `pyproject.toml` for a python project or anything like that, it leaves that entirely up to the python community.

Every type of quality control and metadata standardization should similarly be left up to the relevant community. A layered architecture faciliates this - for example you could create a community-specific public catalog website that only displays entries in the registry if their metadata matches some community standardized schema. That would incentivise data providers in your community to make their metadata compliant, but not block them from sharing it if they don’t.

**Q: Shouldn't we decentralize the storage of the actual data too?**

A: Sure, if you like. It's possible to do that with [OSN](https://www.openstoragenetwork.org/) pods or even cryptographically securely with [IPFS](https://ipfs.tech/). But that's a _separate layer_ from what FROST is concerned with. FROST only catalogs _references_ (i.e. URLs) to where the data exists, and decentralizes the network of records of where the data actually lives. The actual data is stored outside of the network, for example in some organization's S3 bucket. The storage layer is therefore configurable, with the only requirement being that the location of the data and metadata can be expressed as a single public URL. 

### License

[Apache 2.0](https://github.com/TomNicholas/FROST/blob/main/LICENSE)
