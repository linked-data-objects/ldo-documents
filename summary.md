# Linked Data Objects

## Introduction

### Problem being solved - Decoupling of hyperlink target addresses from server names
The goal of Linked Data Objects is to enable storage, retrieval, transmission, and processing of linked data elements (graph nodes) using persistent *href* identifiers that do not change as the storage location is changed.

### Use Case Requirements
The use case requirements include being able to move data location at will, being able to replicate data across multiple storage locations, and being able to use persistent hyperlinks to refer to data elements independent of storage location.

### What is a Linked Data Object?
A Linked Data Object (LDO) is a collection of RDF triples that describes, and optionally includes, some information. An example LDO is a FOAF subject and a set of relevant triples. Another example is a JSON-LD context file. An LDO may be an RDF wrapper that describes a resource in some other content format such as an image file or hardware device API.

An LDO is an atomically validated element, and may be cryptographically signed to ensure end to end integrity through storage, retrieval, transmission, and caching processes. Partial updates to LDOs may also be signed and validated before being applied, an authoritative server may return a selected partial representation of an LDO, and an authoritative server may update the signature of an LDO based on a partial update.

## LDO architecture and technology
An LDO is a straightforward extension of the Linked Data Platform (LDP) that adds a persistent identifier and cryptographic encapsulation to a set of linked data (RDF) triples.

### LDO format
* LDO identifier (UUID)

* LDO context

* LDO triples

* LDO signature

### LDO references
References to an LDO may use the location independent LDO URI scheme, or a location specific scheme that identifies a particular server location which may have been obtained from a directory or other LDO address resolver/router.

Using the LDO URI scheme:

    ldo:b352aea8-ace8-412c-9d56-1c042685726a

On the wire routable, using an http server discovered at 192.168.3.11:33821

    http://192.168.3.11:33821/b352aea8-ace8-412c-9d56-1c042685726a

Reference to component inside LDO using hyperlink:

    ldo:b352aea8-ace8-412c-9d56-1c042685726a/sensors/temperature/currentValue


## LDO prototype
The LDO prototype is implemented as a set of microservices

### Storage
implements persistent storage and retrieval for LDO elements, store and retrieve whole objects

### Cache
implements ephemeral storage for LDO and items of other content formats, allows fine-grain access to items inside LDO

### Reactor
container for scripts that operate on cached elements in response to notifications from the web-proxy, cache, and from other scripts

### Web-proxy
validates http connections from external clients, applies web ACLs, and may map ldo content to html+css

### Typical flow - Cache miss and fill with retry, reactor script invocation
1) Request arrives at web-proxy and ACL is validated, request re-coded and sent to cache 

2) Cache processes requests and finds entry not present, sends lookup request to storage, asks web-proxy to retry

3) Storage indexes the entry from its LDO identifier and returns the object to cache

4) Cache validates the signature of the entry and loads into memory

5) Web-proxy retries cache lookup 

6) Cache processes request and finds asynchronous RESThook
- Cache returns result to web-proxy
- Cache invokes script in reactor

7) Web-proxy returns result to web client
- (reactor processes script)
