# IIIF Resource Discovery

This approach to the discovery of IIIF Resources uses standard, semantic technologies for easy optimization of crawling and indexing processes.  The core technology choice, as agreed upon at the Toronto Working Groups meeting, is the W3C's [ActivityStreams 2.0](https://www.w3.org/TR/activitystreams-core/) specification.


## The Approach

Following the [design principles](http://iiif.io/api/annex/notes/design_patterns/) of IIIF:
 * As Simple as Possible, and No Simpler
 * Intelligently Manage Ramping Up
 * Avoid Specific Technology implementations
 * Follow Resource-Oriented Design
 * Follow Linked Data
 * Design for JSON-LD First
 * Use Standards
 * Follow Best Practices
 * Loosely couple APIs
 * Define Success, not Failure

And the useful patterns established by the [ResourceSync](http://openarchives.org/rs/1.1/resourcesync) effort:
  * List of Resources
  * List of Changes
  * Notification of Changes

As scoped by the [IIIF Discovery charter](http://iiif.io/community/groups/discovery/charter/) and [use cases](https://github.com/iiif/iiif-stories/issues?utf8=%E2%9C%93&q=is%3Aissue%20is%3Aopen%20label%3Adiscovery), we have designed an approach towards solving the challenges faced by the community.

## Activity Streams

[Activity Streams](https://www.w3.org/TR/activitystreams-core/) (AS2) is a "model for representing potential and completed activities". IIIF already uses the AS2 Collections and Paging model, as does the [Web Annotation](https://www.w3.org/TR/annotation-model/#collections) work that we depend on.  AS2 is designed to be JSON-LD first while still respecting the Linked Open Data paradigm, and follows the best practices defined in the W3C.

### List of Resources

The top level resource for managing the lists of IIIF resources is an AS2 Collection. This model is the same as the Web Annotation model for Annotation Collections and Annotation Pages.  AS2 Collections for lists of this size will always be paged, and the Collection document a pointer to the first and last pages, along with metadata about the entire list.

Therefore, the Collection does not directly contain any of the activities or resources, instead it refers to the `first` page of such a list.  The pages are ordered both from page to page by following `next` relationships, and internally within the page in the `items` property. The number of entries in each page is up to the implementer, and cannot be requested by the client.

```json-doc
{
  "@context": [
    "http://iiif.io/api/presentation/3/context.json",
    "https://www.w3.org/ns/activitystreams"],
  "id": "https://example.org/iiif/discovery.json",
  "type": "Collection",
  "label": "Example Big Collection",
  "total": 33000,
  "first": {
    "id": "https://example.org/iiif/discovery-1.json",
    "type": "CollectionPage"
  },
  "last": {
    "id": "https://data.getty.edu/iiif/discovery-10.json",
    "type": "CollectionPage"
  }
}
```

#### Level 0: Basic Resource List

The basic information required, in order to provide a minimally effective set of links to Manifests to harvest is just the URIs of the Manifests. However, with the addition of just a little boilerplate in the JSON, we can be on the path towards a robust set of metadata that allows clients to optimize their harvesting.  

Starting with the Manifest links, we add an "Update" activity wrapper around the URIs.  The order of the Manifests in the pages is unimportant, but each should only appear once. In terms of optimization, it provides no additional benefit over any other simpler list format, but is compatible with a system where optimization is possible, such as the following levels.  This is the minimum level for interoperability, but further levels are significant improvements in terms of efficiency.

__Usage:__  Crawl the entire set of pages and dereference each manifest to see if it has changed.

```json-doc
{
  "@context": [
    "http://iiif.io/api/presentation/3/context.json",
    "https://www.w3.org/ns/activitystreams"],
  "id": "https://example.org/iiif/discovery-1.json",
  "type": "CollectionPage",
  "partOf": {
    "id": "https://example.org/iiif/discovery.json",
    "type": "Collection"
  },
  "next": {
    "id": "https://example.org/iiif/discovery-2.json",
    "type": "CollectionPage"
  },
  "items": [
    {
      "type": "Update",
      "object": {
        "id": "https://example.org/iiif/museum/1/manifest.json",
        "type": "Manifest",
      }
    },
    {
      "type": "Update",
      "object": {
        "id": "https://example.org/iiif/museum/2/manifest.json",
        "type": "Manifest"
      }
    }
  ]
}
```


#### Level 1: Basic Change List

The most effective information to add is the datestamp at which the resource was last modified (including the initial modification that created it).  If we know these dates, we can add them to the activities and order the list such that the most recent activities occur last. The timestamp is given in the `endTime` property -- the time at which the document update process finished. It is up to the implementer to decide whether the update process includes the publication online, or only the internal data modification, but the decision MUST be consistently applied.  

The rationale for crawling backwards is that the first pages, once finished, become static resources.  If the list were ordered from most recent to least, then either the pages would change their URIs, reducing the ability to cache them or the first page would be constantly changing size rather than the last page, which is more familiar.

Note that the Manifests MAY appear multiple times in the list, or only the most recent change might appear, depending on the implementation.

__Usage:__ Record each time the list is crawled. Start from the `last` page and work backwards through the list until a datestamp in `endTime` is encountered before the previous time a crawl occurred, and has thus already been processed.

```json-doc
{
  "@context": [
    "http://iiif.io/api/presentation/3/context.json",
    "https://www.w3.org/ns/activitystreams"],
  "id": "https://example.org/iiif/discovery-1.json",
  "type": "CollectionPage",
  "partOf": {
    "id": "https://example.org/iiif/discovery.json",
    "type": "Collection"
  },
  "next": {
    "id": "https://data.getty.edu/iiif/discovery-2.json",
    "type": "CollectionPage"
  },
  "items": [
    {
      "type": "Update",
      "object": {
        "id": "https://data.getty.edu/iiif/museum/1876/manifest.json",
        "type": "Manifest"
      },
      "endTime": "2017-09-20T00:00:00Z"
    },
    {
      "type": "Update",
      "object": {
        "id": "https://data.getty.edu/iiif/museum/33092/manifest.json",
        "type": "Manifest"
      },
      "endTime": "2017-09-21T00:00:00Z"
    }
    // ...    
  ]
}
```


#### Level 1b: Change List with Metadata

Additional metadata can be added to the basic change list without affecting the overall crawl behavior.  This metadata can include:

* An identifier for the activity, so it can be referenced externally, given in `id`.
* The actor that performed the activity, given in `actor`.
* The application, tool or service used to perform the activity, given in `instrument`.
* The datestamp of when the activity started, given in `startTime`.
* And any additional information about the Manifest, from the Presentation API.

Valuable information about the Manifest, for discovery purposes, includes:

* A `label` to display to the end-user of a discovery system
* A `thumbnail` to display to the end-user of a discovery system
* A `seeAlso` reference to additional metadata that would allow a discovery system to provide a richer or more detailed experience.
* A `within` link to a Collection that that Manifest is part of for context.

__Usage__: Record each time the list is crawled. Start from the `last` page and work backwards through the list until a datestamp before the previous time a crawl occurred is encountered.

```json-doc
{
  "@context": [
    "http://iiif.io/api/presentation/3/context.json",
    "https://www.w3.org/ns/activitystreams"],
  "id": "https://example.org/iiif/discovery-1.json",
  "type": "CollectionPage",
  "partOf": {
    "id": "https://example.org/iiif/discovery.json",
    "type": "Collection"
  },
  "next": {
    "id": "https://data.getty.edu/iiif/discovery-2.json",
    "type": "CollectionPage"
  },
  "items": [
    {
      "id": "http://data.getty.edu/iiif/discovery/event/196421",
      "type": "Update",
      "actor": {
        "id": "http://example.org/museum",
        "type": "Organization"
      },
      "instrument": "http://github.com/example_org/iiif_museum_transformer",
      "object": {
        "id": "https://data.getty.edu/iiif/museum/33092/manifest.json",
        "type": "Manifest",
        "label": {"en": ["Example Museum Object"]},
        "within": "https://example.org/iiif/museum/collection/paintings.json"
      },
      "startTime": "2017-09-19T20:00:00Z",
      "endTime": "2017-09-19T20:01:00Z"
    }
  ]
}
```

#### Level 2: Complete Change List

At the most complex level, a log of all of the activities that have taken place can be recorded, with multiple events per resource.  This would include Deletes, allowing a synchronization process to remove resources as well as add them. This would also allow for the complete history of a resource to be reconstructed, if each version has an archived representation.  The list might end up very long if there are many changes to resources, however this is not a typical situation as far as we know, and the cost is still not terrible as each entry is short and can be compressed both on disk and at the HTTP(S) transport layer.

The core activity types are:

* `Create`: The initial creation of the resource.  Each resource must have at most one Create event in which it is the `object`.
* `Update`: Any change to the resource.  In a system that does not distinguish creation from modification, then all changes MAY be `Update`s. 
* `Delete`: The deletion of the resource, or its de-publication from the web. 

Additional activities which institutions may also find helpful are listed [in this annex](activities.md/).

__Usage:__  Record each time the list is crawled. Start from the `last` page and work backwards through the list until a datestamp before the previous time a crawl occurred is encountered. Only process the most recent change per resource, including deleting resources.

```json-doc
{
  "@context": [
    "http://iiif.io/api/presentation/3/context.json",
    "https://www.w3.org/ns/activitystreams"],
  "id": "https://example.org/iiif/discovery-1.json",
  "type": "CollectionPage",
  "partOf": {
    "id": "https://example.org/iiif/discovery.json",
    "type": "Collection"
  },
  "next": {
    "id": "https://example.org/iiif/discovery-2.json",
    "type": "CollectionPage"
  },
  "items": [
    {
      "id": "http://data.getty.edu/iiif/discovery/event/196421",
      "type": "Delete",
      "object": {
        "id": "https://data.getty.edu/iiif/museum/1/manifest.json",
        "type": "Manifest"
      },
      "endTime": "2017-09-25T20:00:01Z"
    },
    {
      "id": "http://data.getty.edu/iiif/discovery/event/100001",
      "type": "Update",
      "object": {
        "id": "https://data.getty.edu/iiif/museum/1/manifest.json",
        "type": "Manifest"
      },
      "endTime": "2017-09-19T20:01:00Z"
    },
    {
      "id": "http://data.getty.edu/iiif/discovery/event/54627",
      "type": "Create",
      "object": {
        "id": "https://data.getty.edu/iiif/museum/1/manifest.json",
        "type": "Manifest"
      },
      "endTime": "2017-06-07T08:00:30Z"
    }    
  ]
}
```

## Processing Algorithm

The aim of the processing algorithm is to inform harvesters how to make best use of the available information, with an aim of finding "indexable content" -- the descriptive information that might be used to build an index of the resources to allow them to be discovered.  For different types of resource, and for different domains, the "indexable content" will have different formats and semantics. At worst, the data in the Manifest and other IIIF resources might be used, despite its presentational intent. 

__Collection Algorithm__

Given the URI of an ActivityStreams Collection (`collection`) as input, a conforming processor SHOULD:

1. Initialization:
  1. Let `processedItems` be an empty array
  2. Let `lastCrawl` be the timestamp of the previous time the algorithm was executed
2. Retrieve the representation of `collection` via HTTP(S)
3. Minimally validate that it conforms to the specification
4. Find the URI of the last page at `collection.last.id` (`pageN`)
5. Apply the results of the page algorithm to `pageN`


__Page Algorithm__

Given the URI of an ActivityStreams CollectionPage (`page`) and the date of last crawling (`lastCrawl`) as input, a conforming processor SHOULD:

1. Retrieve the representation of `page` via HTTP(S)
2. Minimally validate that it conforms to the specification
3. Find the set of updates of the page at `page.items` (`items`)
4. In reverse order, iterate through the activities (`activity`) in `items`
  1. For each `activity`, if `activity.endTime` is before `lastCrawl`, then terminate ;
  2. If the updated resource's uri at `activity.target.id` is in `processedItems`, then continue ;
  3. Otherwise, if `activity.type` is `Update` or `Create`, then find the URI of the updated resource at `activity.target.id` (`target`) and apply the target resource algorithm ;
  4. Otherwise, if `activity.type` is `Delete`, then find the URI of the deleted resource at `activity.target.id` and remove it from the index.
  5. Add the processed resource's URI to `processedItems`
5. Finally, find the URI of the previous page at `collection.prev.id` (`pageN1`)
6. Apply the results of the page algorithm to `pageN1`

__Target Resource Algorithm__

Given the URI of a target resource (`target`), a conforming processor SHOULD:

1. Retrieve the representation of `target` via HTTP(S)
2. Minimally validate that it conforms to the appropriate specification
3. Find the URI of the resource at `target.id` (`targetId`)
4. If the resource has the `seeAlso` property, then for each resource referenced ('extref')
  1. If the `format` and/or `profile` of `extref` is understood by the processor, then it should retrieve any representations that it can process to extract Indexable Content.
5. Otherwise, extract any indexable content from the resource's representation, using resource type specific functionality.
6. Index the content against `targetId`


## Ongoing Experiments

* Digirati
  * Endpoint: https://mcgrattan.org/as/
  * Implementation: https://github.com/mattmcgrattan/iiif-activity-streams

* OCLC
  * Endpoint: http://52.204.112.237:3051/activity-streams
  * Implementation: https://github.com/mixterj/iiif_activity_streams_creation
  * Consumer: https://github.com/mixterj/activity_streams_harvest

* NCSU
  * Endpoint: https://d.lib.ncsu.edu/collections/iiif-discovery.json

* IIIF
  * Endpoint: http://preview.iiif.io/root/discovery_list/community/groups/discovery/registry_as.json
