# Representational State Transfer (REST) API Design

* [Guidelines](#guidelines)
	* [Headers](#headers)
		* [Content-Type](#header-content-type)
		* [Accept](#header-accept)
		* [Accept-Language](#header-accept-language)
		* [Date](#header-date)
		* [Location](#header-location)
		* [Cache Headers](#header-cache)
		* [Last-Modified](#header-last-modified)
		* [ETag](#header-etag)
		* [Precondition Headers](#header-precondition)
		* [Authorization](#header-authorization)
	* [Resource Naming](#resource-naming)
		* [Collection and Item Pattern](#collection-item-pattern)
		* [Hierarchical Pattern](#hierarchical-pattern)
		* [Creation of Resources and Representations](#creating-resources)
		* [Use of the query component in naming resources](#resource-naming-query)
		* [Providing an array of resources](#resource-array)
		* [Use of the fragment component in identifiers](#fragment-component-use)
		* [Resource Naming Syntax](#resource-naming-syntax)
			* [Example URI Templates](#example-uri-templates)
		* [Friendly resource name pattern](#resource-naming-friendly)
	* [Hypermedia as the Engine of Application State](#hypermedia)
	* [Related Data](#related-data)
	* [Custom Data](#custom-data)
	* [Service Index](#index)
	* [Pagination](#pagination)
	* [Data Design](#data)
		* [Identifiers](#data-identifiers)
		* [Self-describing Data](#data-self-describing)
		* [Dates and Times](#data-date-time)
		* [Currency](#data-currency)
		* [Key-Value Pair Names](#data-key-names)
	* [HTTP Status Codes](#http-status-codes)
		* [Errors when HTTP Status Code is 4xx](#errors-when-4xx)
	* [Documentation](#documentation)
		* [Markdown](#documentation-markdown)
		* [Schema](#documentation-schema)
* [Works Cited](#works-cited)

## <a name="guidelines"></a>Guidelines

### <a name="headers"></a>Headers

* Services SHOULD limit themselves to standards based HTTP headers as defined in the [Internet Assigned Numbers Authority (IANA) Message Headers](http://www.iana.org/assignments/message-headers/message-headers.xhtml) (Protocol=HTTP, Status=Standard)

#### <a name="header-content-type"></a>Content-Type

* Services SHOULD always provide the [RFC 7231 Content-Type](https://tools.ietf.org/html/rfc7231#section-3.1.1.5) header field in all responses.

#### <a name="header-accept"></a>Accept

* Services SHOULD respect the [RFC 7231 Accept](https://tools.ietf.org/html/rfc7231#section-5.3.2) request header field whenever possible.
* Services SHOULD provide a [406 Not Acceptable](https://tools.ietf.org/html/rfc7231#section-6.5.6) HTTP status code when unable to respect the Accept header field.
* Services MAY disregard the Accept header field and treat the response as if it is not subject to content negotiation provided the [Content-Type](#header-content-type) header is present in the response.

#### <a name="header-accept-language"></a>Accept-Language

* Services SHOULD respect the [RFC 7231 Accept-Language](https://tools.ietf.org/html/rfc7231#section-5.3.5) request header field and respond with the appropriately localized data when applicable.
* The Accept-Language request header field uses [RFC 5646 Tags for Identifying Languages](#RFC-5646). Simplest examples are:
	* en-US
	* de-DE
* Services MAY default to US English (en-US) in the absence of the Accept-Language request header field.

#### <a name="header-date"></a>Date

* Services SHOULD provide the [RFC 7231 Date](https://tools.ietf.org/html/rfc7231#section-7.1.1.2) response header field.

#### <a name="header-location"></a>Location

* Services SHOULD provide the [RFC 7231 7.1.2. Location](https://tools.ietf.org/html/rfc7231#section-7.1.2) response header field with the value expressed as a fully qualified domain name (FQDN) [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax](#RFC-3986) for all operations which use the `POST` HTTP method.
* Services MAY provide the Location header as a relative value noting this requires client code to build URIs which should generally be avoided.

#### <a name="header-cache"></a>Cache Headers

For a full understanding of caching see [RFC 7234 Hypertext Transfer Protocol (HTTP/1.1): Caching](https://tools.ietf.org/html/rfc7234). This section along with the [Last-Modified](#header-last-modified), [ETag](#header-etag) and [Precondition](#header-precondition) sections cover the caching responsibilities for a service.

* Services SHOULD help clients with cacheability of responses though the use of headers.
	* [RFC 7234 Age](https://tools.ietf.org/html/rfc7234#section-5.1)
	* [RFC 7234 Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2)
		* [RFC 7234 Response Cache-Control Directives](https://tools.ietf.org/html/rfc7234#section-5.2.2)
	* [RFC 7234 Expires](https://tools.ietf.org/html/rfc7234#section-5.3)
	* [RFC 7234 Pragma](https://tools.ietf.org/html/rfc7234#section-5.4)
	* [RFC 7234 Warning](https://tools.ietf.org/html/rfc7234#section-5.5)

#### <a name="header-last-modified"></a>Last-Modified

* Services SHOULD provide the [RFC 7232 Last-Modified](https://tools.ietf.org/html/rfc7232#section-2.2) header in all responses.

#### <a name="header-etag"></a>ETag

* Services SHOULD provide the [RFC 7232 ETag](https://tools.ietf.org/html/rfc7232#section-2.3) header in all responses.

#### <a name="header-precondition"></a>Precondition Headers

* Services SHOULD support the following request header fields:
	* [RFC 7232 If-Match](https://tools.ietf.org/html/rfc7232#section-3.1)
	* [RFC 7232 If-None-Match](https://tools.ietf.org/html/rfc7232#section-3.2)
	* [RFC 7232 If-Modified-Since](https://tools.ietf.org/html/rfc7232#section-3.3)
	* [RFC 7232 If-Unmodified-Since](https://tools.ietf.org/html/rfc7232#section-3.4)
	* [RFC 7232 If-Range](https://tools.ietf.org/html/rfc7232#section-3.5)
* Services SHOULD respond with the following HTTP status codes to conditional requests:
	* [304 Not Modified](https://tools.ietf.org/html/rfc7232#section-4.1)
	* [412 Precondition Failed](https://tools.ietf.org/html/rfc7232#section-4.2)

#### <a name="header-authorization"></a>Authorization

* Services which make use of access tokens which contain data (such as [JSON Web Token](./jwt-summary.md)) SHOULD NOT leverage the data therein for any portion of the request other than authorization.

Generally speaking a well designed interface of an API will not be aware of the authorization mechanism being used and this authorization mechanism can be replaced wholesale without affecting any client code whatsoever other than authorization pieces. In other words: Authorization schemes can be changed / replaced and this action only affects the `Authorization` header.

### <a name="resource-naming"></a>Resource Naming

The following is a direct copy+paste from [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax](#RFC-3986) as a baseline for the balance of this section.

```
  foo://example.com:8042/over/there?name=ferret#nose
  \_/   \______________/\_________/ \_________/ \__/
   |           |            |            |        |
scheme     authority       path        query   fragment
```

* All portions of a URI SHOULD be in lowercase.
* Included in the path in the following order:
	* Services SHOULD include their name in the first URI path segment.
		* Service names SHOULD be alphanumeric characters only.
		* Service names SHOULD NOT include `service` in their name.
		* Service names SHOULD NOT include punctuation.
	* Services SHOULD provide a [Service Index](#index) at this root path.
	* Services MAY include a version number in the URI path using the form of vX where X is a positive integer: `v1`, `v2`, ... , `v10` in the second URI path segment.
		* Generally speaking, well designed services should never need a version number. It is included here for completeness.
		* For an excellent talk on why "no versioning" is a good thing fast forward to 27:50 in [GOTO 2014 • REST: I don't Think it Means What You Think it Does](https://youtu.be/pspy1H6A3FM?t=1670).
		* Versioning overload makes it difficult to independently evolve services.
	* Service MAY include a resource name.
	* Services MAY include a resource identifier.
	* Services MAY include a representation name.
* Services SHOULD have unique names.
	* `/stores`
	* `/aisles`
* Services SHOULD NOT share paths.
	* Examples:
		* `/stores/aisles` and `/stores/products` is better expressed as distinct services: `/stores`, `/aisles`, `/products` and `/inventory`.
	* Reasons:
		* Service(s) would have to implement a reverse proxy at each hop.
		* Versioning overload makes it difficult to independently evolve services.
		* Service(s) would have to manage top level disaster recovery where an API gateway (reverse gateway) can handle.
		* It breaks the consistency model of `{service}/{collection}/{item}?representation={value}`.

#### <a name="collection-item-pattern"></a>Collection and Item Pattern

* Services SHOULD arrange and name resources according to a `/collection/item` paradigm.
  * `collection` name is plural.
  * `item` name is singular.
* Services SHOULD have a base collection, the root of which is the [Service Index](#index) resource.
* The base collection MAY contain individual items.
* The base collection MAY contain additional collections.

```
https://example.com/stores                                            // The base collection.
https://example.com/stores/76cc758e256c438b8e49546e0102b8c8           // A single item within the collection.

https://example.com/stores/schemas                                    // A related collection within the service.
https://example.com/stores/schemas/com-example-store.schema.json      // A single item within the collection.
```

#### <a name="hierarchical-pattern"></a>Hierarchical Pattern

> The [Collection and Item Pattern](#collection-item-pattern) is preferred.

* Services MAY use paths which describe a parent-child hierarchy of resources available within the domain of the service.
* This pattern:
  * Can cause resources to be identified in an overly complex way.
  * Can reveal / leak implementation details like data storage paradigms, business logic, business unit seams and more.
  * Can more easily lead to Remote Procedure Call (RPC) architecture style.
  * Usually duplicates data which should be or already is present in the resource itself.
* This pattern MUST NOT be a substitute for proper handling of [Hypermedia as the Engine of Application State](#links-hateoas) or [Related Data](#related-data).

```
Template: https://example.com/{service}/{version}/{identifier}{child}/{identifier}.../{child}/{identifier}
Example:  https://example.com/stores/v1/76cc758e256c438b8e49546e0102b8c8/aisles/df62491e95f54fe1a3ef9bdf40c4d1f5/shelves/c66f06fdb31b4882ad995e4d19ca7aed
```

#### <a name="creating-resources"></a>Creation of Resources and Representations

> One example (of many approaches) for resources, representations and related data can be found in [Resources and Representations](./resource-and-representation.md).

* Services SHOULD use the `query` component of [RFC 3986 Uniform Resource Identifier (URI)](#RFC-3986) to denote secondary representations.
* Services MAY create as many representations as is needed.
  * Services are encouraged to do so in order to logically order the representations.
  * Services are encouraged to do so to avoid overcomplicating paths.
* Services MAY make the same representation available via multiple paths.
* Services MAY make subsets of resources available in representations.

##### <a name="creating-resources-example"></a>Examples

**Primary Representation**

URI: `https://example.com/stores/76cc758e256c438b8e49546e0102b8c8`

```json
{
  "href": "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8",
  "id": "76cc758e256c438b8e49546e0102b8c8",
  "template": "https://example.com/stores/{id}",
  "schema": "https://example.com/schemas/com-example-store-2018-03-01.schema.json",
  "name": "Alpha",
  "phone": "(425) 555-1212",
  "operations": [
    {
      "rel": "add-aisle",
      "href": "https://example.com/stores/aisles",
      "method": "POST"
    }
  ]
}
```

**Secondary Representation**

* A representation containing only the metadata and one key value pair, excluding all other data and the HATEOAS operations.
* A pattern of `{key}={value}` has been used in naming: `76cc758e256c438b8e49546e0102b8c8?representation=phone`

URI: `https://example.com/stores/76cc758e256c438b8e49546e0102b8c8?representation=phone`

```json
{
  "href": "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8?representation=phone",
  "id": "76cc758e256c438b8e49546e0102b8c8",
  "template": "https://example.com/stores/{id}?representation=phone",
  "schema": "https://example.com/schemas/com-example-store-2018-03-01.schema.json",
  "phone": "(425) 555-1212"
}
```

#### <a name="resource-array"></a>Providing an array of resources

* A resource which provides multiple items from the collection.
* A naming pattern of `collection/item` is still present, the item returned is an array.

URI: `https://example.com/stores/all`

```json
[
  {
    "href": "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8",
    "id": "76cc758e256c438b8e49546e0102b8c8",
    "template": "https://example.com/stores/{id}",
    "schema": "https://example.com/schemas/com-example-store-2018-03-01.schema.json",
    "name": "Alpha",
    "phone": "(425) 555-1212",
    "operations": [
      {
        "rel": "add-aisle",
        "href": "https://example.com/stores/aisles",
        "method": "POST"
      }
    ]
  }
]
```

#### <a name="resource-naming-query"></a>Use of the query component in naming resources

* Services MAY use the query component of a URI as non-hierarchical data to identify resources.
* Services SHOULD NOT change the nature of a resource in response to the presence of the query component.
* Services MAY change the representation provided for a resource in response to the presence of the query component.

##### <a name="resource-naming-query-example"></a>Examples

```
https://example.com/things?userId=foo          // Array of things for a user.
https://example.com/things?companyId=baz       // Array of things for a company.
https://example.com/things/123?userId=foo      // Single item with user query component.
https://example.com/things/123?companyId=baz   // Single item with company query component.
```

#### <a name="fragment-component-use"></a>Use of the fragment component in identifiers

From [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax Section 3.5 Fragment](https://tools.ietf.org/html/rfc3986#section-3.5):

> "Fragment identifiers have a special role in information retrieval systems as the primary form of client-side indirect referencing, allowing an author to specifically identify aspects of an existing resource that are only indirectly provided by the resource owner.  As such, the fragment identifier is not used in the scheme-specific processing of a URI; instead, the fragment identifier is separated from the rest of the URI prior to a dereference, and thus the identifying information within the fragment itself is dereferenced solely by the user agent, regardless of the URI scheme."

This means the server will not typically receive the fragment portion of the URI in any requests and if it does receive a fragment it should ignore the data.

To demonstrate this use `cUrl` with a URI which contains a fragment -- the tool will remove the fragment portion before sending.

```
curl -v https://retrosight.github.io/rest/hateoas-model-example.html#start

> GET /rest/hateoas-model-example.html <-- fragment removed
> Host: retrosight.github.io
```

#### <a name="resource-naming-syntax"></a>Resource Naming Syntax

Building upon everything in this section the following illustrates how teams should think of URIs when naming resources:

```
                     service  collection     resource identifier     representation identifier
                        |        |                   |                    |
                       _|__   ___|__   ______________|_______________   __|________________
                      /    \ /      \ /                              \ /                   \
  https://example.com/stores/fixtures/71b1d7acbb254e05b7f9060b0a29efab?representation=digest
  \___/   \_________/ \______________________________________________/ \___________________/
    |          |                                  |                              |
 scheme    authority                             path                          query
```

* Naming paradigms and patterns for the `path` are optional and flexible.
* The distinction in the following URI templates is subtle: `{collection}` is also an `{identifier}`.
* Typically (but not always) the resource is an array when `{collection}` and a non-array (single item) when `{identifier}`.

##### <a name="example-uri-templates"></a>Example URI Templates

```
https://example.com/stores                                                   // Base collection (typically the service index)
https://example.com/stores?{key}={value}                                     // Base collection with query component
https://example.com/stores?representation={value}                            // Representation of the base collection
https://example.com/stores/{collection}                                      // Collection within the service
https://example.com/stores/{collection}?{key}={value}                        // Collection within the service with query component
https://example.com/stores/{collection}?representation={value}               // Representation of a collection
https://example.com/stores/{identifier}                                      // Single item within the base collection
https://example.com/stores/{identifier}?{key}={value}                        // Single item with query component
https://example.com/stores/{identifier}?representation={value}               // Representation of a single item within the base collection
https://example.com/stores/{collection}/{identifier}                         // Single item within a collection
https://example.com/stores/{collection}/{identifier}?{key}={value}           // Single item within a collection with query component
https://example.com/stores/{collection}/{identifier}?representation={value}  // Representation of a single item within a collection
```

#### <a name="resource-naming-friendly"></a>Friendly resource name pattern

There are times when resource names should have friendly names readable by humans, for example:

* When the client code chooses the resource name with a `PUT` operation creating a new resource.
* When the resource is more widely used across client code, for example: JSON Schema.

In order to avoid name collisions a pattern MAY be agreed upon and it is suggested that pattern be, in part, based on how [RFC 1034 Domain Names - Concepts and Facilities](#RFC-1034) works:

```
{generic top-level domain (gTLD)}
 -{domain}
  -{second and lower level domains separated with a -}
   -{schema name}
    -{version}
      .schema
        .json

Examples:

  com-example-baseball-equipment-glove-2018-03-01.schema.json
  org-example-soccer-worldcup-2018-39f6256e.schema.json
```

* Services SHOULD refrain from using dots / periods (`.`) in resource names.
  * Approaching friendly names from the perspective of a file based system may come a desire to append a file name to the end a resource name, for example: `com-example-baseball-equipment-glove-2018-03-01.schema.json`. This may be shortsighted and may appear to be in conflict with the [RFC 7231 Content-Type](https://tools.ietf.org/html/rfc7231#section-3.1.1.5) header should a server decide to make multiple formats available in the future.

### <a name="search"></a> Search

> A primer on Search can be found [here](./search.md).

* Services SHOULD provide a dedicated endpoint for the purposes of finding resources within all of the service.
* Services MAY provide search within collections.
* It may be helpful to think of search as another resource collection.

```
https:/example.com/stores/search?product=milk
https:/example.com/stores/fixtures/search?type=shelf
```

### <a name="hypermedia"></a>Hypermedia as the Engine of Application State

> A primer on Hypermedia as the Engine of Application State can be found [here](./hateoas-model-example.md).

> The differences between [Hypermedia as the Engine of Application State](#hypermedia), [Related Data](#related-data) and [Identifiers](#data-identifiers) are explained [here](./id-related-data-hateoas.md).

* APIs SHOULD expose the full application state to the consumers
* Services SHOULD make links available only when they are valid for the state of the service at the time of the response.
* Hypermedia as the Engine of Application State describes what client code can do with the server.
* The "application state" portion of the concept refers to the server (not the client).
* It is encapsulated within an `operations` array.
* Client code binds to the `rel` (relation) key and follows the link in the `href` key using the HTTP method in the `method` key.
  * This insulates against breaking changes as it allows the server to change the `href` value at any time and the client code will still work.

#### <a name="hypermedia-operations"></a>Operations Schema

Name | Type | Format | Description
-----|------|--------|------------
`operations`|`array`|[`operation`](#hypermedia-operations-operation)|The hypermedia as the engine of application state (HATEOAS) information.

#### <a name="hypermedia-operations-operation"></a>Operation Schema

Name | Type | Format | Description
-----|------|--------|------------
`rel`|`string`|[RFC 5988 Relation Type](https://tools.ietf.org/html/rfc5988#section-4)|**Required** Relation type as defined by the server. There are registered relation types listed in [RFC 5988 6.2.2. Initial Registry Contents](https://tools.ietf.org/html/rfc5988#section-6.2.2) including pagination relation types of `next`, `prev`, `first` and `last`.
`href`|`string`|[RFC 3986 Uniform Resource Identifier (URI)](#RFC-3986)|**Required** Hyperlink to the resource. This key name is borrowed from the HTML5 `href` element and behaves similarly.
`method`|`string`|[RFC 7231 Methods](https://tools.ietf.org/html/rfc7231#section-4.3)|Default is GET when `method` is not specified. Valid method names are RFC 7231 GET, HEAD, POST, PUT, DELETE, CONNECT, OPTIONS, TRACE + RFC 5789 PATCH.

#### Example

* See the [Service Index](#index) or [Pagination](#pagination) sections for examples.

### <a name="related-data"></a>Related Data

> The differences between [Hypermedia as the Engine of Application State](#hypermedia), [Related Data](#related-data) and [Identifiers](#data-identifiers) are explained [here](./id-related-data-hateoas.md).

> One example (of many approaches) for resources, representations and related data can be found in [Resources and Representations](./resource-and-representation.md).

* All relationships to data outside of the document SHOULD be described by fully qualified domain name (FQDN) `URI` resource links.
* It is acceptable for data to reference other data which does not have a corresponding reference back; In the example given it would be fine if the products did not have a link to the store.
* Related data can reference resources outside of the domain.

#### Example

In the following example:
* `products` is the items available for sale in a store.
* `aisles` is an array of aisles for the store.
* `map` is a hyperlink to the location for the store within a external domain.

Contents of https://example.com/stores/abc:

```json
{
  "name":"Example",
  "products": "https://example.com/products/123",
  "aisles": [
    "https://example.com/aisles/1",
    "https://example.com/aisles/2"
  ],
  "map": "https://www.google.com/maps/place/1820+Wensley+Dr,+Charlotte,+NC+28210/@35.1541824,-80.8694037,17z"
}
```

### <a name="custom-data"></a>Custom Data

* Services SHOULD NOT use ambiguous key names such as `custom1`.
* Services SHOULD use a [RFC 3986 Uniform Resource Identifier (URI)](#RFC-3986) to disambiguate data and avoid name collisions.

Name | Type | Format | Description
-----|------|--------|------------
`customData`|`array`|-|An array of `object` each of which is disambiguated with a URI to a schema.

```json
{
  "comment": "The customData key would be in a larger JSON document.",
  "customData": [
    {
      "schema":"https://example.com/schema/be9e17e751bd4dc690ff3d079c8808fe",
      "foo": "hello",
      "baz": "world"
    },
    {
      "schema":"https://example.com/schema/someotheridentifier",
      "alpha": "bravo",
      "delta": "tango",
      "echo": "foxtrot"
    }
  ]
}
```

### <a name="index"></a>Service Index

> More information on the service index design and schema can be found in [Service Index](./service-index.md).

* Services SHOULD provide a service index of links at the base URI which will be interesting to callers of the service.
* The links SHOULD follow the [RFC 5988 Web Linking](#RFC-5988) standard.
* The links MAY use [RFC 6570 URI Template](#RFC-6570).
* The service index SHOULD be made available at the root of the service: `https://example.com/{servicename}`.

The service index is one of the keys to evolvability of services, allowing client code to:

* Reference an abstracted name rather than a link directly which allows service links to change without requiring client code changes.
* Simply follow given links rather than using `string` builders or concatenation to craft service links.

### <a name="pagination"></a>Pagination

* Services SHOULD always determine the pagination scheme rather than allowing client code to define (for example, with a query string parameter) to allow for service design and performance tuning independent of client code.
* Services SHOULD provide pagination operations in all responses using the following relation names according to the [RFC 5988 Web Linking](#RFC-5988) standard:
	* `next` - refers to the next resource in a ordered series of resources.
	* `prev` - refers to the previous resource in an ordered series of resources.
* Services MAY also use the following relation names as desired:
	* `first` - refers to the furthest preceding resource in a series of resources..
	* `last` - refers to the furthest following resource in a series of resources.
* Services MAY use any sort of name to denote pages of data. `page` is acceptable as would be any other name which makes sense internally to the service itself. Naming is of less importance here because client code simply follows the links rather than crafting URIs.
* For more information see [Pagination Design](/pagination-design.md)

#### Schema

Pagination leverages the [Hypermedia as the Engine of Application State](#hypermedia) [`operations`](#hypermedia-operations) schema.

#### Example

```json
{
  "data": [
    { "id": "Alpha" },
    { "id": "Bravo" },
    { "id": "Charlie" }
  ],
  "operations": [
    {
      "rel": "next",
      "href": "https://example.com/service/page4"
    },
    {
      "rel": "prev",
      "href": "https://example.com/service/page2"
    },
    {
      "rel": "first",
      "href": "https://example.com/service/first"
    }
    {
      "rel": "last",
      "href": "https://example.com/service/last"
    }
  ]
}
```

### <a name="data"></a>Data Design

#### <a name="data-identifiers"></a>Identifiers

> The differences between [Hypermedia as the Engine of Application State](#hypermedia), [Related Data](#related-data) and [Identifiers](#data-identifiers) are explained [here](./id-related-data-hateoas.md).

* Services SHOULD identify unique resources with a string in the format of a [RFC 4122 A Universally Unique IDentifier (UUID) URN Namespace](#RFC-4122) UUID4 without dashes in a key named `id`.
	* This value SHOULD be separate from database key values (i.e., primary key) to avoid leaking implementation details.
* Services SHOULD provide a URI as the identifier (including the UUID4 value stored in `id`) for the resource in a key named `href`.
	* The key name `href` is based on the [HTML5 concept of links](https://www.w3.org/TR/html5/links.html), [RFC 5988 Web Linking Appendix A Notes on Using the Link Header with the HTML4 Format](https://tools.ietf.org/html/rfc5988#appendix-A) and is consistent with other sections of the guidelines where hyperlinks are provided using the Hypermedia as the Engine of Application State [Operations Schema](#hypermedia-operations-operation) such as [Related Data](#related-data), [Service Index](#index) and [Pagination](#pagination).
* Services SHOULD provide a [RFC 6570 URI Template](#RFC-6570) in a key named `template`.
	* The combination of `template` + `id` = `href` allows legacy relational database systems the flexibility to store URI values minimally to avoid database bloat (mainly due to indexing) and therefore storage costs.
* `href`, `id` and `template` SHOULD be considered reserved keywords.
  * `href` and `id` SHOULD only be used in the root of a representation document.
  * `template` SHOULD be used as a key name only when the value is a URI template.

##### Example

```json
{
  "href": "https://example.com/service/ae7d9679708f48e2951bbefd478b3d16",
  "id": "ae7d9679708f48e2951bbefd478b3d16",
  "template": "https://example.com/service/{id}",
  "comment": "...and additional resource data follows."
}
```

#### <a name="data-self-describing"></a>Self-describing Data

* Representations SHOULD be self-describing through the use of a `schema` key in the root of all documents.
* The `schema` value SHOULD be in the form of a [RFC 3986 Uniform Resource Identifier (URI)](#RFC-3986).
* Examples of schema illustrating the paradigms found in this document can be found in [schema](./schema).

```json
{
  "schema": "https://example.com/schema/com-example-base-2018-03-01.schema.json"
}
```

#### <a name="data-date-time"></a>Dates and Times

* Services SHOULD choose from the following formats to accept and represent dates and timestamps:
  * **Preferred**: [RFC 3339 Date and Time on the Internet: Timestamps](#RFC-3339): `YYYY-MM-DDThh:mm:ss.nnn-hh:mmZ`
  * [ISO 8601:2004](#ISO-8601) UTC + Offset: `YYYY-MM-DDThh:mm:ss.nnn-hhmmZ`
  * [Unix Time](https://en.wikipedia.org/wiki/Unix_time) also known as `POSIX` and `Epoch` time.

##### Examples

```json
RFC 3339
{
  "keyName": "2016-10-18T03:56:06.798-07:00",
  "keyName": "2016-10-18T03:56:06.798Z"
}

ISO 8601
{
  "keyName": "2016-10-18T03:56:06.798-0700",
  "keyName": "2016-10-18T03:56:06.798Z"
}

Unix
{
  "keyName": 1477246370
}
```

#### <a name="data-currency"></a>Currency

* All currency values SHOULD follow [ISO 4217:2015 Codes for the representation of currencies](#ISO-4217).
* A currency value SHOULD be an JSON object with the following keys:
  * `amount`
    * Stored as `string` data type due to poor handling of floating point numbers in Javascript.
    * Only valid numbers.
    * Optional single decimal point.
    * Optional negative (-) sign.
    * The value SHOULD NOT contain formatting:
      * No monetary symbols.
      * No thousands separator.
  * `currency` - The [ISO 3166](#ISO-3166) country code.
* Schema: [./schema/com-example-currencyamount-2018-03-01.schema.json](./schema/com-example-currencyamount-2018-03-01.schema.json)

```json
{
  "subtotal": {
    "amount": "-1234.56",
    "currency": "USD"
  },
  "tax": {
    "amount": "1.00",
    "currency": "CHF"
  },
  "total": {
    "amount": "100",
    "currency": "JPY"
  }
}
```

#### <a name="data-key-names"></a>Key-Value Pair Names

* Services SHOULD use [Camel Case](https://en.wikipedia.org/wiki/CamelCase) with the first letter in lower case for all key-value pair names. Example: `total`, `currencyCode` and `createDate`.

### <a name="http-status-codes"></a>HTTP Status Codes

* Services SHOULD always use the appropriate HTTP status codes as defined in [RFC 7231 Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](https://tools.ietf.org/html/rfc7231).

The following table summarizes the status code ranges and is a direct copy from the aforementioned RFC.

Status Code Range|Definition
-----------------|----------
1xx|The 1xx (Informational) class of status code indicates an interim response for communicating connection status or request progress prior to completing the requested action and sending a final response.
2xx|The 2xx (Successful) class of status code indicates that the client's request was successfully received, understood, and accepted.
3xx|The 3xx (Redirection) class of status code indicates that further action needs to be taken by the user agent in order to fulfill the request.
4xx|The 4xx (Client Error) class of status code indicates that the client seems to have erred.
5xx|The 5xx (Server Error) class of status code indicates that the server is aware that it has erred or is incapable of performing the requested method.

#### <a name="errors-when-4xx"></a>Errors when HTTP Status Code is 4xx

* Services SHOULD provide additional context via JSON entity document when 4xx HTTP status code is provided in the response.
* Generally speaking, most 4xx errors occur occur during a `PUT` or `POST` operation.
* There may not be a need for returning 4xx errors for a service that is primarily `GET` operations noting there are exceptions for `GET` operations, for example:
	* A `GET` should return a 4xx when there are invalid parameters in the query string.
	* A 404 Not Found should be returned when the `GET` URI is dynamic and a value is wrongly formatted, or does not exist, or the user does not have permission.
* The schema starts with an array which allows the services to expand the items within the errors without breaking the contract of the service itself. It is expected many services will only ever return a single item in the array.

##### <a name="errors-when-4xx-errors"></a>Errors Schema

Name | Type | Format | Description
-----|------|--------|------------
`errors`|`array`|[`error`](#errors-when-4xx-error)|**Required** An array of errors.

##### <a name="errors-when-4xx-error"></a>Error Schema
Name | Type | Format | Description
-----|------|--------|------------
`errorCode`|`string`|-|**Required** Machine readable code associated with the error which is static and never localized. Examples: `dateTimeMissing`, `OutOfMem` and `invalidUser`. These could also be UUID4 (`a1d7bb3bb19348b0858687acc9e303ec`), number (`123456`) or a URI (`https://example.com/errors/invaliduser`). Whatever form is chosen it's worth noting contextual strings are helpful to developers reading the code.
`errorMessage`|`string`|-|**Required** Message associated with the error.
`dataPath`|`string`|-|Relative data path.
`schemaPath`|`string`|-|Relative schema path.
`errors`|`array`|[`error`](#errors-when-4xx-error)|An array of errors. Note: this points to this schema as errors can nest.

##### Examples

**Minimum**

```
HTTP/1.1 400 Bad Request
Date: Tue, 19 Jul 2016 18:23:16 GMT
```

```json
{
  "errors": [
    {
      "errorCode": "missingRequiredProp",
      "errorMessage": "Missing required property: dateTime"
    }
  ]
}
```

**Non-nested**

```
HTTP/1.1 400 Bad Request
Date: Tue, 19 Jul 2016 18:23:16 GMT
```

```json
{
  "errors": [
    {
      "errorCode": "missingRequiredProp",
      "errorMessage": "Missing required property: dateTime"
    },
    {
      "errorCode": "parseDataFailed",
      "errorMessage": "Unable to parse data."
    }
  ]
}
```

**Nested**

* Also included in this example are the optional keys.

```
HTTP/1.1 400 Bad Request
Date: Tue, 19 Jul 2016 18:23:16 GMT
```

```json
{
  "errors": [
    {
      "errorCode": "missingRequiredProp",
      "errorMessage": "Missing required property: dateTime",
      "dataPath": "/merchant/location/address",
      "schemaPath": "/allOf/0/required/0",
      "errors": [
        {
          "errorCode": "parseDataFailed",
          "errorMessage": "Unable to parse data.",
          "dataPath": "/merchant/location/address/state",
          "schemaPath": "/allOf/0/required/0/0"
        }
      ]
    }
  ]
}
```

### <a name="documentation"></a>Documentation

#### <a name="documentation-markdown"></a>Markdown

* Services SHOULD fully document the data structures, all verbs and how the service works using [Markdown](https://daringfireball.net/projects/markdown/syntax) including all of the sections within the [Documentation Template](./documentation-template.md).

#### <a name="documentation-schema"></a>Schema

* Services SHOULD fully document data structures using [JSON schema](#json-schema) whether or not the schema is used for validation purposes within the service.
* Services SHOULD make the schema available via a source code repository.
* Services MAY make the schema available via the service itself.

Title|$id|Local|Source
---|---|---|---
Core schema meta-schema (v7)|`http://json-schema.org/draft-07/schema#`|[org-json-schema-schema-v7.json](./schema/org-json-schema-schema-v7.json)|[Source](https://github.com/json-schema-org/json-schema-spec/blob/draft-07/schema.json)
JSON Hyper-Schema (v7)|`http://json-schema.org/draft-07/hyper-schema#`|[org-json-schema-hyper-schema-v7.json](./schema/org-json-schema-hyper-schema-v7.json)|[Source](https://github.com/json-schema-org/json-schema-spec/blob/draft-07/hyper-schema.json)
JSON Hyper-Schema Output (v7)|`http://json-schema.org/draft-7/hyper-schema-output`|[org-json-schema-hyper-schema-output-v7.json](./schema/org-json-schema-hyper-schema-output-v7.json)|[Source](https://github.com/json-schema-org/json-schema-spec/blob/draft-07/hyper-schema-output.json)
Link Description Object (v7)|`http://json-schema.org/draft-07/links#`|[org-json-schema-links-v7.json](./schema/org-json-schema-links-v7.json)|[Source](https://github.com/json-schema-org/json-schema-spec/blob/draft-07/links.json)

## <a name="works-cited"></a>Works Cited

### <a name="rest"></a>Representational State Transfer (REST) Dissertation by Roy Fielding, 2000

* [https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)

Also included inline are the references to the parts of the dissertation outside of chapter 5.

* [5.1 Deriving REST](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1)
	* [5.1.1 Starting with the Null Style](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_1)
	* [5.1.2 Client-Server](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_2)
		* Reference: [3.4.1 Client-Server (CS)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_1)
	* [5.1.3 Stateless](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_3)
		* Reference: [3.4.3 Client-Stateless-Server (CSS)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_3)
	* [5.1.4 Cache](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_4)
		* Reference: [3.4.4 Client-Cache-Stateless-Server (C$SS)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_4)
	* [5.1.5 Uniform Interface](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_5)
	* [5.1.6 Layered System](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_6)
		* Reference: [3.4.2 Layered System (LS) and Layered-Client-Server (LCS)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_2)
	* [5.1.7 Code-On-Demand](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_7)
		* Reference: [3.5.3 Code on Demand (COD)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_5_3)
	* [5.1.8 Style Derivation Summary](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_8)
* [5.2 REST Architectural Elements](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2)
	* [5.2.1 Data Elements](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_1)
		* [5.2.1.1 Resources and Resource Identifiers](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_1_1)
		* [5.2.1.2 Representations](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_1_2)
	* [5.2.2 Connectors](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_2)
	* [5.2.3 Components](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_3)
* [5.3 REST Architectural Views](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_3)
	* [5.3.1 Process View](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_3_1)
	* [5.3.2 Connector View](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_3_2)
	* [5.3.3 Data View](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_3_3)
* [5.4 Related Work](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_4)
* [5.5 Summary](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_5)

### <a name="hateoas"></a>Hypermedia as the Engine of Application State

* [https://en.wikipedia.org/wiki/HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)

### <a name="RFC-7230-7237"></a>RFC 7230-7237 Hypertext Transfer Protocol (HTTP/1.1)

* [RFC 7230 Hypertext Transfer Protocol (HTTP/1.1): Message Syntax and Routing](https://tools.ietf.org/html/rfc7230)
* [RFC 7231 Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](https://tools.ietf.org/html/rfc7231)
	* [GET](https://tools.ietf.org/html/rfc7231#section-4.3.1)
	* [HEAD](https://tools.ietf.org/html/rfc7231#section-4.3.2)
	* [POST](https://tools.ietf.org/html/rfc7231#section-4.3.3)
	* [PUT](https://tools.ietf.org/html/rfc7231#section-4.3.4)
	* [DELETE](https://tools.ietf.org/html/rfc7231#section-4.3.5)
	* [CONNECT](https://tools.ietf.org/html/rfc7231#section-4.3.6)
	* [OPTIONS](https://tools.ietf.org/html/rfc7231#section-4.3.7)
	* [TRACE](https://tools.ietf.org/html/rfc7231#section-4.3.8)
	* [Request Header Fields](https://tools.ietf.org/html/rfc7231#section-5)
	* [Response Status Codes](https://tools.ietf.org/html/rfc7231#section-6)
	* [Response Header Fields](https://tools.ietf.org/html/rfc7231#section-7)
* [RFC 7232 Hypertext Transfer Protocol (HTTP/1.1): Conditional Requests](https://tools.ietf.org/html/rfc7232)
* [RFC 7233 Hypertext Transfer Protocol (HTTP/1.1): Range Requests](https://tools.ietf.org/html/rfc7233)
* [RFC 7234 Hypertext Transfer Protocol (HTTP/1.1): Caching](https://tools.ietf.org/html/rfc7234)
* [RFC 7235 Hypertext Transfer Protocol (HTTP/1.1): Authentication](https://tools.ietf.org/html/rfc7235)
* [RFC 7236 Initial Hypertext Transfer Protocol (HTTP) Authentication Scheme Registrations](https://tools.ietf.org/html/rfc7236)
* [RFC 7237 Initial Hypertext Transfer Protocol (HTTP) Method Registrations](https://tools.ietf.org/html/rfc7237)

### <a name="RFC-5789"></a>RFC 5789 PATCH Method for HTTP

* [https://tools.ietf.org/html/rfc5789](https://tools.ietf.org/html/rfc5789)

### <a name="RFC-5646"></a>RFC 5646 Tags for Identifying Languages

* [https://tools.ietf.org/html/rfc5646](https://tools.ietf.org/html/rfc5646)

### <a name="json"></a>Javascript Object Notation (JSON)

* [http://www.json.org](http://www.json.org)
* [http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf](http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf)
* [https://tools.ietf.org/html/rfc7159](https://tools.ietf.org/html/rfc7159)

### <a name="json=patch"></a>JavaScript Object Notation (JSON) Patch

* [https://tools.ietf.org/html/rfc6902](https://tools.ietf.org/html/rfc6902)

### <a name="json-schema"></a>JSON Schema

* [http://json-schema.org](http://json-schema.org)

### <a name="RFC-7519"></a>RFC 7519 JSON Web Token (JWT)

* [https://tools.ietf.org/html/rfc7519](https://tools.ietf.org/html/rfc7519)

Links to claims within the RFC:

* [4.1 Registered Claim Names](https://tools.ietf.org/html/rfc7519#section-4.1)
	* [4.1.1 "iss" (Issuer) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.1)
	* [4.1.2 "sub" (Subject) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.2)
	* [4.1.3 "aud" (Audience) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.3)
	* [4.1.4 "exp" (Expiration Time) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.4)
	* [4.1.5 "nbf" (Not Before) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.5)
	* [4.1.6 "iat" (Issued At) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.6)
	* [4.1.7 "jti" (JWT ID) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.7)

#### Additional Resources

* [JWT Summary](./jwt-summary.md)
* [https://jwt.io](https://jwt.io)
* [Public Claim Names](http://www.iana.org/assignments/jwt/jwt.txt)

### <a name="RFC-3339"></a>RFC 3339 Date and Time on the Internet: Timestamps

* [https://tools.ietf.org/html/rfc3339](https://tools.ietf.org/html/rfc3339)

### <a name="RFC-5280"></a>RFC 5280 Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile

* [https://tools.ietf.org/html/rfc5280](https://tools.ietf.org/html/rfc5280)

### <a name="RFC-6749"></a>RFC 6749 OAuth 2.0 Authorization Framework

* [https://tools.ietf.org/html/rfc6749](https://tools.ietf.org/html/rfc6749)

### <a name="RFC-1034"></a>RFC 1034 Domain Names - Concepts and Facilities

* [https://tools.ietf.org/html/rfc1034](https://tools.ietf.org/html/rfc1034)

### <a name="RFC-3986"></a>RFC 3986 Uniform Resource Identifier (URI): Generic Syntax

* [https://tools.ietf.org/html/rfc3986](https://tools.ietf.org/html/rfc3986)

### <a name="RFC-4122"></a>RFC 4122 A Universally Unique IDentifier (UUID) URN Namespace

* [https://tools.ietf.org/html/rfc4122](https://tools.ietf.org/html/rfc4122)
* Services SHOULD use all lowercase with dashes preserved.
	* This RFC generally recommends a lower case when producing values (while allowing for uppercase) and requires case-insensitivity when parsing.
* For more information on UUID4 see [4.4. Algorithms for Creating a UUID from Truly Random or Pseudo-Random Numbers](https://tools.ietf.org/html/rfc4122#section-4.4).

### <a name="RFC-5988"></a>RFC 5988 Web Linking

* [https://tools.ietf.org/html/rfc5988](https://tools.ietf.org/html/rfc5988)

### <a name="RFC-6570"></a>RFC 6570 URI Template

* [https://tools.ietf.org/html/rfc6570](https://tools.ietf.org/html/rfc6570)

### <a name="IANA-headers"></a>Internet Assigned Numbers Authority (IANA) Message Headers

* [http://www.iana.org/assignments/message-headers/message-headers.xhtml](http://www.iana.org/assignments/message-headers/message-headers.xhtml)

### <a name="ISO-3166"></a>ISO 3166-1 Country Codes

* Online Browsing Platform: [https://www.iso.org/obp/ui/#search/code/](https://www.iso.org/obp/ui/#search/code/)
* [http://www.iso.org/iso/home/standards/country_codes.htm](http://www.iso.org/iso/home/standards/country_codes.htm)

### <a name="ISO-4217"></a>ISO 4217:2015 Codes for the representation of currencies

* [ISO 4217:2015 Codes for the representation of currencies](https://www.iso.org/standard/64758.html)

### <a name="ISO-8601"></a>ISO 8601:2004 Data elements and interchange formats -- Information interchange -- Representation of dates and times

* [https://www.iso.org/standard/40874.html](https://www.iso.org/standard/40874.html)

### ECMAScript® 2017 Language Specification (ECMA-262, 8th edition, June 2017)

* [https://www.ecma-international.org/ecma-262/8.0/index.html](https://www.ecma-international.org/ecma-262/8.0/index.html)

### <a name="RFC-2119"></a>RFC 2119 Key words for use in RFCs to Indicate Requirement Levels

* [https://www.ietf.org/rfc/rfc2119.txt](https://www.ietf.org/rfc/rfc2119.txt)
* All requirements are noted with uppercase (MUST, SHOULD) in bulleted lists to disambiguate from narrative prose using lower case (should, must).

The following table has been added for ease of understanding and is a direct copy of the definitions contained in RFC 2119.

Keyword|Synonyms|Definition
-------|--------|----------
MUST|Required, Shall|Absolute requirement.
MUST NOT|Shall Not|Absolute prohibition.
SHOULD|Recommended|There may exist valid reasons in particular circumstances to **ignore a particular item**, but the full implications must be understood and carefully weighed before choosing a different course.
SHOULD NOT|Not Recommended|There may exist valid reasons in particular circumstances when the particular **behavior is acceptable** or even useful, but the full implications should be understood and the case carefully weighed before implementing any behavior described with this label.
MAY|Optional|An item is truly optional
