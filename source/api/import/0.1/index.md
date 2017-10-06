---
title: "IIIF Import API 0.1.0"
title_override: "IIIF Import API 0.1.0"
id: import-api
layout: spec
tags: [specifications, import-api]
major: 0
minor: 1
patch: 0
pre: draft
cssversion: 2
sitemap: false
---

## Status of this Document
{:.no_toc}
__This Version:__ {{ page.major }}.{{ page.minor }}.{{ page.patch }}{% if page.pre != 'final' %}-{{ page.pre }}{% endif %}

{% include beta.md %}

**Editors**

  * **[Michael Appleby](https://orcid.org/0000-0002-1266-298X)** [![ORCID iD](/img/orcid_16x16.png)](https://orcid.org/0000-0002-1266-298X), [_Yale University_](http://www.yale.edu/)
  * **[Tom Crane](https://orcid.org/0000-0003-1881-243X)** [![ORCID iD](/img/orcid_16x16.png)](https://orcid.org/0000-0003-1881-243X), [_Digirati_](http://digirati.com/)
  * **[Robert Sanderson](https://orcid.org/0000-0003-4441-6852)** [![ORCID iD](/img/orcid_16x16.png)](https://orcid.org/0000-0003-4441-6852), [_Stanford University_](http://www.stanford.edu/)
  * **[Jon Stroop](https://orcid.org/0000-0002-0367-1243)** [![ORCID iD](/img/orcid_16x16.png)](https://orcid.org/0000-0002-0367-1243), [_Princeton University Library_](https://library.princeton.edu/)
  * **[Simeon Warner](https://orcid.org/0000-0002-7970-7855)** [![ORCID iD](/img/orcid_16x16.png)](https://orcid.org/0000-0002-7970-7855), [_Cornell University_](https://www.cornell.edu/)
  {: .names}

{% include copyright2017.md %}

## Table of Contents
{:.no_toc}

* Table of Discontent (will be replaced by macro)
{:toc}

## 1. Introduction

The IIIF (pronounced "Triple-Eye-Eff") [Presentation API][prezi-api] allows content to be brought together from distributed systems via annotations.  That content might include images, often with a IIIF [Image API][image-api] service to access them, audio, video, rich or plain text, or anything else.  There is a rich ecosystem of [IIIF viewing clients][iiif-clients] with different affordances to meet different user needs.  This specification describes approaches that allow users to easily import IIIF resources into the viewing client they choose.

Please join in discussion of this draft specification on the [IIIF Discovery Technical Specification Group][iiif-discovery-tsg] calls, or send feedback to [iiif-discuss@googlegroups.com][iiif-discuss]

### 1.1. Use Cases

Use cases for import of IIIF resources to viewers include:

 * User has an multi-item IIIF viewer application open. In another browser window, the user is browsing or using a search interface, and locates an image, book, manuscript, etc. that they want to add to their open viewer.
 * User has an IIIF application open that provides the ability to edit a IIIF [Presentation API][prezi-api] Manifest. In another browser window, the user is browsing or using a search interface, and locates an image that they want to add to the manifest they are creating.

FIXME - Want to have 3-5 key pithy use cases in here that motivate the set of patterns proposed.
{: .warning}

This is WORK IN PROGRESS: Additional use cases may be submitted on [github][iiif-discovery-use-cases] for discussion by the [IIIF Discovery Technical Specification Group][iiif-discovery-tsg].
{: .warning}

### 1.2. Terminology

The key words _MUST_, _MUST NOT_, _REQUIRED_, _SHALL_, _SHALL NOT_, _SHOULD_, _SHOULD NOT_, _RECOMMENDED_, _MAY_, and _OPTIONAL_ in this document are to be interpreted as described in [RFC 2119][rfc-2119].

## 2. Overview

FIXME - Motivation for UI patterns and scope, and description elements and concepts that are commons across patterns.
{: .warning}

## 3. Open in Viewer

FIXME - need a pattern, see [#99](https://github.com/IIIF/iiif-stories/issues/99), [#90](https://github.com/IIIF/iiif-stories/issues/90), [#85](https://github.com/IIIF/iiif-stories/issues/85).
{: .warning}

## 4. Add to Viewer

### 4.1 Drag and Drop

[Drag and drop][drag-and-drop] is a common pattern in graphical user interfaces that is supported by most windowing systems and web browsers. It requires a recognizable item to select or "grab", and a location to "drag" to before "dropping". This specification uses a hyperlinked icon to support drag and drop of an IIIF resource from one web page to an IIIF viewing application in another web page. Common web browser implementations support this pattern by copying the hyperlink target URI as payload of the drop action. An example HTML snippet is shown below:

```
<a href="http://example.org/help?manifest=http://example.org/manifest1">
  <img src="iiif-dragndrop-100px.png" alt="Drag item to IIIF viewer"/>
</a>
```

The three components to this snippet: link target, link parameters, icon are described in the following sections.

FIXME - Should probably rework this to use [HTML5 Drag and drop][html5-dnd].
{: .warning}

#### 4.1.1 Link Target

A user following the hyperlink will be directed to the specified URI (`http://example.org/help?manifest=http://example.org/manifest1` in the example above). It is _RECOMMENDED_ that this resolve to a page that explains the drag and drop facility, or opens a particular viewer showing the image or manifest.

FIXME - Is there community agreement on this behavior?
{: .warning}

#### 4.1.2 Link Parameters

The query parameters of the link target (`manifest=http://example.org/manifest1`) provide the information necessary for the IIIF viewing application receiving the "drop" to open or add the specified resource. The query parameters _MAY_ include:

  * `manifest` - URI of an IIIF [Presentation API][prezi-api] Manifest (`http://example.org/manifest1` in the example snippet).
  * `canvas` - URI of currently selected Canvas within the Manifest. If no Canvas is specified then the target application will use its default behavior to present the Manifest. (The `canvas` parameter has no meaning unless a `manifest` parameter is given.)
  * `image` - URI of an IIIF [Image API][image-api] `info.json`. This is an alternative to specifying a Manifest for use in situations where the source implements only the Image API. It _SHOULD NOT_ be specified in addition to `manifest` or `canvas`.

#### 4.1.3 Icon

It is _RECOMMENDED_ that the following icon be used as a recognizable way to indicate an IIIF resource that may be dragged following the drag and drop pattern described in this section.

![icon](iiif-dragndrop-100px.png)

This icon may be freely reused according to the terms of [CC0][cc0]. The icon _MAY_ be scaled to fit in with other user interface design elements.

FIXME - Need a real icon. The IIIF logo has been used in a number of implementations.
{: .warning}

### 4.2 Copy Paste

FIXME - Work out copy paste pattern.
{: .warning}

## Appendices

### A. Versioning

This specification follows [Semantic Versioning][semver]. See the note [Versioning of APIs][versioning] for details regarding how this is implemented.

### B. Acknowledgements

Many thanks to the members of the [IIIF][iiif-community] for their engagement, ideas and feedback.

### C. Change Log

| Date       | Description                                        |
| ---------- | -------------------------------------------------- |
| 2017-09-07 | Version 0.1.0 draft of drag-and-drop pattern only  | 
| 2015-09-30 | Trial implementation of [drag-and-drop at IIIF Shimathon][shimathon-drag-and-drop] |
{: .api-table}


[cc-by]: http://creativecommons.org/licenses/by/4.0/ "Creative Commons &mdash; Attribution 4.0 International"
[iiif-discuss]: mailto:iiif-discuss@googlegroups.com "Email Discussion List"
[iiif-discovery-tsg]: http://iiif.io/community/groups/discovery/ "IIIF Discovery Technical Specification Group"
[iiif-discovery-use-cases]: https://github.com/IIIF/iiif-stories/issues?q=is%3Aissue+is%3Aopen+label%3Adiscovery-import "IIIF Discovery Import Use Cases"
[iiif-clients]: http://iiif.io/apps-demos/#implementation-demos "IIIF Viewing Clients"
[versioning]: /api/annex/notes/semver/ "Versioning of APIs"
[semver]: http://semver.org/spec/v2.0.0.html "Semantic Versioning 2.0.0"
[iiif-community]: /community/ "IIIF Community"

[image-api]: /api/image/2.0/ "Image API"
[prezi-api]: /api/presentation/2.0/ "Presentation API"
[rfc-2119]: http://tools.ietf.org/html/rfc2119

[drag-and-drop]: https://en.wikipedia.org/wiki/Drag_and_drop "Drag and Drop"
[shimathon-drag-and-drop]: https://zimeon.github.io/iiif-dragndrop/ "IIIF Drag-and-Drop Experiments"
[cc0]: https://creativecommons.org/publicdomain/zero/1.0/ "CC0 1.0 Universal (CC0 1.0) Public Domain Dedication"
[html5-dnd]: https://www.w3.org/TR/html/editing.html#drag-and-drop "HTML5 Drag and Drop"

{% include acronyms.md %}
