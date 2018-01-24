# IIIF Resource Discovery: Activities

This approach to the discovery of IIIF Resources uses standard, semantic technologies for easy optimization of crawling and indexing processes.  The core technology choice, as agreed upon at the Toronto Working Groups meeting, is the W3C's [ActivityStreams 2.0](https://www.w3.org/TR/activitystreams-core/) specification.

## Types of Activity

### Single-Resource Publisher Operations

#### Create

Level: 1

The object resource came into existence. Before this time it did not exist, and the state of the resource is the first state.

__Use Case:__ As a content publisher, I want to alert search engines that I created a new Manifest, so that they can harvest and index it.

```json
{
	"id": "http://example.org/iiif/discovery/event/54627",
	"type": "Create",
	"object": {
		"id": "https://example.org/iiif/museum/1/manifest.json",
		"type": "Manifest"
	},
	"endTime": "2017-06-08T00:00:00Z"
}
```


#### Update / Modify

Level: 0

A resource that was already created was changed, or was created and the activity is published by a system that cannot distinguish modification from creation. Any previous state may also be available at another URI.

__Use Case:__ As a content publisher, I want to alert search engines that I updated a Manifest, so that they can re-harvest and update their indexes.

```json
{
	"id": "http://example.org/iiif/discovery/event/100001",
	"type": "Update",
	"object": {
		"id": "https://example.org/iiif/museum/1/manifest.json",
		"type": "Manifest"
	},
	"endTime": "2017-09-19T20:00:00Z"
}
```

#### Delete

Level: 2

A resource that was already created was taken out of existence. After this time, the resource does not exist, and previous states may be available at other URIs.

__Use Case:__ As a content publisher, I want to alert search engines that I deleted a Manifest, so that they can remove it from their indexes.

```json
{
	"id": "http://example.org/iiif/discovery/event/8172645",
	"type": "Delete",
	"object": {
		"id": "https://example.org/iiif/museum/1/manifest.json",
		"type": "Manifest"
	},
	"endTime": "2017-09-19T20:00:00Z"
}
```


### Multi-Resource Publisher Operations

These operations are compositions of the above in terms of the effects on different resources, but should be treated as a single operation for the purpose of notification and replay.  These are more appropriate for true synchronization systems, rather than simple crawling and discovery.

The advantage to using them is that they frequently do not require the resources to be re-harvested, instead all of the necessary information about the change is included in the activity itself.

#### Add 

Level: 4

The `object` resource was added to the `target`, such as a Manifest being added to a Collection.

For the purposes of discovery systems, the inclusion of the manifest in the collection might affect indexing, if there are per-collection indexes, even if there are no other changes to the Collection or Manifest. Instead of an Update to the Collection, this allows the actual change to be communicated so that the Collection document does not need to be re-harvested.

__Use Case:__ As a content publisher, I want to alert search engines that I added a Manifest to a particular Collection, such that a Collection-specific index can be updated.

```json
{
	"id": "http://example.org/iiif/discovery/event/123462",
	"type": "Add",
	"object": {
		"id": "https://example.org/iiif/museum/1/manifest.json",
		"type": "Manifest"
	},
	"target": {
		"id": "https://example.org/iiif/museum/paintings/collection.json",
		"type": "Collection"
	},
	"endTime": "2017-09-19T20:00:00Z"
}
```

#### Remove

Level: 4

The `object` resource was removed from the `origin` resource, such as removing a Manifest from a Collection.

This is the inverse of `Add` for the indexing management of the membership of Collections. 

__Use Case:__ As a content publisher, I want to alert search engines that I removed a Manifest from a particular Collection, such that a Collection-specific index can be updated.

```json
{
	"id": "http://example.org/iiif/discovery/event/123462",
	"type": "Remove",
	"object": {
		"id": "https://example.org/iiif/museum/1/manifest.json",
		"type": "Manifest"
	},
	"origin": {
		"id": "https://example.org/iiif/museum/paintings/collection.json",
		"type": "Collection"
	},
	"endTime": "2017-09-19T20:00:00Z"
}
```


#### Copy

Minimum Level: 3

The source resource was duplicated to create or modify the target resource.  This activity is not a core ActivityStreams type.

For the purposes of discovery, the object resource's representation can be treated as the target resource's representation, but the object resource should not (yet) be removed from the index. This does not require the target resource to be re-harvested.

__Use Case:__ Needed for Copy to be included. Does anyone actually copy rather than move, under what circumstances, and why?
{: .warning}

```json
{
	"id": "http://example.org/iiif/discovery/event/123462",
	"type": "Copy",
	"object": {
		"id": "https://example.org/iiif/museum/1/manifest.json",
		"type": "Manifest"
	},
	"target": {
		"id": "https://example.org/iiif/publications/102314/manifest.json",
		"type": "Manifest"
	},
	"endTime": "2017-09-19T20:00:00Z"
}
```

#### Rename / Move

Minimum Level: 4

The object resource is duplicated to create or modify the target resource, and the original was deleted as part of the operation.

For the purposes of discovery, the representation of the object resource should now be treated as the representation of the target resource, and the object resource removed from the index.  This does not require the target resource to be reharvested.

__Use Case:__ As a content publisher, I want to alert search engines that I have changed my URI patterns and thereby moved a Manifest to a different URI, such that the indexes can be updated to point to the new location.

```json
{
	"id": "http://example.org/iiif/discovery/event/123462",
	"type": "Move",
	"object": {
		"id": "https://example.org/iiif/museum/1/manifest.json",
		"type": "Manifest"
	},
	"target": {
		"id": "https://example.org/iiif/publications/102314/manifest.json",
		"type": "Manifest"
	},
	"endTime": "2017-09-19T20:00:00Z"
}
```

#### Merge

Minumum Level: 4

Two or more source resources were combined to create or modify the target resource.  This activity is not a core ActivityStreams type.

For the purposes of discovery, as the merge is not necessarily complete, nor is it clear whether or not the merged resources were deleted or modified as part of the process, all of the resources should be reharvested. The user interface for the discovery system might wish to group them together in some way to visualize the merge to the user.

__Use Case:__ Needed. While symmetrical with Split, we need an actual use case for when and why anyone would merge resources.
{: .warning}

```json
{
	"id": "http://example.org/iiif/discovery/event/123462",
	"type": "Merge",
	"object": [
		{
			"id": "https://example.org/iiif/museum/1/manifest.json",
			"type": "Manifest"
		},
		{
			"id": "https://example.org/iiif/museum/2/manifest.json",
			"type": "Manifest"
		}
	],
	"target": {
		"id": "https://example.org/iiif/museum/1726/manifest.json",
		"type": "Manifest"
	},
	"endTime": "2017-09-19T20:00:00Z"
}
```

#### Split

Minumum Level: 4

A single source resource was divided to create or modify multiple target resources.

For the purposes of discovery, as the split is not necessarily complete, nor is it clear whether or not the original resource was deleted or modified as part of the process, all of the resources should be reharvested.

__Use Case:__ As a content publisher, I want to alert search engines that I split a Manifest representing a large archive into smaller pieces, where each piece better represents an object, and thus for search engines to reharvest all of the pieces. Note that there is no distinction between Split and Delete+Create+Create as everything needs to be harvested. Better use case required.
{: .warning}


```json
{
	"id": "http://example.org/iiif/discovery/event/123462",
	"type": "Split",
	"object": {
		"id": "https://example.org/iiif/museum/1726/manifest.json",
		"type": "Manifest"
	},
	"target": [
		{
			"id": "https://example.org/iiif/museum/1/manifest.json",
			"type": "Manifest"
		},
		{
			"id": "https://example.org/iiif/museum/2/manifest.json",
			"type": "Manifest"
		}
	],
	"endTime": "2017-09-19T20:00:00Z"
}
```

### Third Party Operations

These operations are carried out by third parties, rather than the publisher.  As such they would be written to the Inbox of the resource as either a request to modify the resource, or just notification that the resource was used.


#### Reference

Minimum Level: 3

A remote source resource has a new reference or link to the target local resource.

```json
{
	"id": "http://example.org/iiif/discovery/event/8172645",
	"type": "Reference",
	"object": {
		"id": "https://example.edu/iiif/9/manifest.json",
		"type": "Manifest"
	},
	"target": {
		"id": "https://example.org/iiif/museum/1/manifest.json",
		"type": "Manifest"
	},
	"endTime": "2017-09-19T20:00:00Z"
}
```


#### Use

Minimum Level: 3

The target resource was used by the remote agent in some way.

```json
{
	"id": "http://example.org/iiif/discovery/event/8172645",
	"type": "Use",
	"object": {
		"id": "https://example.org/iiif/museum/1/manifest.json",
		"type": "Manifest"
	},
	"endTime": "2017-09-19T20:00:00Z"
}
```

