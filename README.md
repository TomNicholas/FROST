# FROST (Federated Registry Of Scientific Things)

FROST is a decentralized subscribable data catalog protocol for sharing all scientific data globally.

![updating](https://github.com/user-attachments/assets/2e864f8f-7a29-4ea7-88a7-6b2365071869)

(See [the FAQ](https://github.com/TomNicholas/FROST?tab=readme-ov-file#faqs) for explanation of the GIF)

> [!WARNING]  
> This doesn't actually exist yet, this repo is just for brainstorming ideas. Please contribute!

## Motivation

**Context:**
The best way to store and provide access to big scientific datasets is via [ARCO data](https://medium.com/pangeo/step-by-step-guide-to-building-a-big-data-portal-e262af1c2977) in S3-compatible cloud object storage. We now have scalable cloud-optimised formats that are version-controlled at rest in object storage (particularly [Icechunk](https://icechunk.io/) for arrays and [Iceberg](https://iceberg.apache.org/) for tables). This is _huge_, as even dynamically-updated datasets can now be distributed via raw S3, with no other server needed. All the data providers who are paying attention are about to put their data in these formats, and then they will try to advertise the S3 URLs to the world via ad-hoc data catalogs.

**Problem: ⛓️‍💥 Everyone's catalogs are disconnected from everyone else's ⛓️‍💥**. 

This means:
- No cross-org discoverability (e.g. NASA catalog users won't see NOAA datasets or vice versa).
- No cross-org tracking of updates (e.g. NOAA datasets derived directly from NASA datasets won't automatically know if the NASA datasets have been updated upstream).
- Risk of "catalog wars" where platform services compete to make more and more comprehensive "meta-catalogs" which merely track (outdated) links to other orgs' data.
- Risk that if one platform does win everyone might feel locked in to it via the social network effect.

**Solution: Federated catalog protocol with cross-org publish-subcribe model.** 

- Cross-org discoverability enabled via displaying the contents of the dataset entries being broadcast,
- Cross-org tracking of updates to datasets enabled the same way,
- No need to compete to make a better catalog, as anyone can easily consume and display the entire global catalog, including updates,
- Federated trust model allows proliferation of high-quality centralized services, whilst also guarding against platform lock-in.

**How do we build it?:** Not sure exactly, but the problem is analogous to creating Federated alternatives to centralized social media (i.e. Bluesky/Mastodon vs Twitter). Perhaps we can piggyback off of Bluesky's ATproto or Mastodon's ActivityPub?

> [!NOTE]  
> See also the [motivating blog post](https://hackmd.io/@TomNicholas/H1KzoYrPJe), and the [presentation on FROST](https://discourse.pangeo.io/t/pangeo-showcase-frost-federated-registry-of-scientific-things-feb-12-2025/4861) at the Pangeo showcase.

## Concept

- Decentralized network protocol for disseminating updates to version-controlled datasets

## Idea

- Global catalog of version-controlled Icechunk/Iceberg etc. datasets
- Shared as a decentralized / federated network spanning different organisations, so that no one organisation controls the catalog
- Publish updates to catalog entries, which then propagate through the network between organisations
- Allows datasets to subscribe to changes in upstream datasets
- Doesn’t store or transfer any data, just sends updates about which parts of it have changed

## Inspiration

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

## Goals

- Version-controlled
- Subscribable
- Searchable
- Decentralized (or at least Federated)

## Non-goals

- Data storage
- Implementing the version-controlled data model
- Domain-specific features (e.g. recording spatio-temporal coverage of a dataset)
- Social network features beyond subscribing (e.g. commenting, liking)
- Doing any loading/computation/anything else once update is received

## Technical requirements

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

## Protocol Implementation

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

## Use cases

Once such a protocol is in place, many different use cases / business models / services become easier to build.

### Public Dataset Catalog

Data provider organisations could publish their datasets and be confident that anyone interested can programmatically find their data and track their updates. They can still build a catalog showing just their own org's datasets, but they can also broadcast their data offerings in a way that is easy for other organisations to track. (We distinguish between the global _registry_ of all datasets and updates, and various _catalogs_, which are subsets of the registry of interest to one organisation or community.)

### Search Engine

As anyone could consume the "firehose" of public dataset updates, anyone could build a website which filters or queries that entires in any way before displaying them. In particular they could experiment with different models of search, independent of how the actual registry data is disseminated. The simplest example would be keyword-based search (e.g. NOAA could provide a page that displays all datasets across any org, but only those with "ocean" in the metadata), but the same architecture would also allow for more complex semantic search services using ML techniques.

### Data Lake

A platform that provides additional services on top of an organisation's private data is known as a _data lake_. In addition to public cataloging features, the federated protocol would allow datasets in data lakes to recieve and act upon updates from public datasets in other organisations, even if the downstream datasets in the data lake were not publicly exposed.

### Data Marketplace

A data marketplace is basically just a data catalog but with one extra layer between the registry and the storage - an access control layer which grants access to the raw data only upon authenticated payment. This allows for an entirely different business model with almost exactly the same architecture. Sellers of data could broadcast their offerings globally, and if some measure of price was included in the registry schema, their prices could automatically be displayed in anyone else's catalogs. As the price in the catalog need not be the actual price paid upon negotiating with the data provider, the resulting experience would be somewhat like using Facebook Marketplace, where the listed price is only intended as a rough expectation, and actual transactions occur outside of the FROST network.

### Mass Backups

With all links to public datasets made available, anyone could easily find and suck out all the datasets they considered important into a replica as a backup.

### Real-time data services

Updates to datasets should propagate through the network automatically and quickly (in seconds to minutes using ATproto). This enables real-time data services to be built upon the data sharing network, e.g. recomputing wildfire risk each time new satellite imagery becomes available.

## FAQ’s

**Q: That GIF is pretty, but what does it mean?**

A: The GIF is intended to show notifications of dataset updates propagating through a federated network. 

Each node is a version-controlled dataset sitting in S3, in either Icechunk or Iceberg format. The datasets are spread across 3 organisations: NASA, NOAA, and a startup (rocketship). Although each dataset sits in the owning organisation's object storage, the locations, versions, and dependencies of each are shared publicly via the FROST protocol. They thus form a cross-org (federated) network, the FROST network.

A source of new data (the satellite) causes the NASA dataset to be updated. A notification of this update is broadcast to it's dependent datasets. A re-computation of these dependents is triggered, and updated versions of each written out.

**Q: Where are the computations which create the new versions of each dataset running?**

A: Not within the FROST network - the compute task deployments are deliberately separate. Those spinning cogs in the GIF are just meant to indicate some task running somewhere that was triggered by a notification sent via FROST, and will publish an update to FROST once the task is complete. (The tasks could even be manual - i.e. a human receives a notification telling them to look at the updated upstream data before deciding how to update their derived dataset.) The compute layer is an area where platform providers could innovate and compete - they could provide something like Github Actions but for automatically updating datasets instead of updating codebases. But their solutions should be quite general - being too prescriptive as to how these computations must be done will discourage people from using the network.

**Q: Why is Icechunk/Iceberg such a big deal?**

A: [Icechunk](https://icechunk.io/) is the biggest thing since [Zarr](https://zarr.dev/). In case you've been living under a rock, or more sympathetically if “Serverless ACID Transactional Array Database” doesn’t mean anything to you (it didn’t to me when I first read it either), let me summarize the implications:
- Icechunk allows scientific data providers like NASA to make updates, fixes, and additions to their public datasets whenever they like,
- Without disrupting their users access to the data for even one millisecond, no matter how many scientists are using the data at that moment,
- The users can look back in time to see what the data looked like in the past (which is crucial if you ever want to reproduce someone else’s data analysis),
- The Icechunk code can be used by anyone, at any scale, for free, forever, and the format is open so there's no risk of losing access to data,
- It doesn't require running any server or additional service layer above S3,
- It's specifically designed to handle scientific array data (e.g. climate model outputs) and stream the data to you efficiently with all the power of Zarr,
- On top of of that when combined with [VirtualiZarr](https://icechunk.io/icechunk-python/virtual/) you can add all these features to existing data archives of data in pre-cloud formats without making a duplicate copy of PetaBytes of data.

Icechunk is directly inspired by [Apache Iceberg](https://iceberg.apache.org/) (and some similar formats like Delta Tables), which is the same thing but for tabular data.

**Q: Why does the network need to span across different fields of science?**

A: Any attempt to split up the catalog by fields of science will inevitably divide some community’s interdisciplinary field in two. Imagine if github only let you put code for Neuroscience analysis in it. That's fine at the labelling level, but not cool to force that community to bridge two separate networks to receive all updates they care about. Your data probably isn’t that special anyway, you almost certainly could fit it into this framework.

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
