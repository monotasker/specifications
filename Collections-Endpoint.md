# Collection Endpoint

The Collection endpoint is used for accessing and editing metadata on textual resources and their relationships. For DTS purposes, a "resource" is any entity with a textual representation: a letter, novel, manuscript, inscription, play, interview transcript, etc. A set of related resources is called a "collection". A collection consists of a set of members along with common metadata that describes those members. Each member of a collection **must** be either a resource or another collection. Collections can be nested to an arbitrary depth, though a collection does not necessarily need to have any children (i.e., the collection may be empty).

## The Collections Endpoint and Document Structures

Note that a resource need not represent a complete work. One might choose to represent a literary work as a collection, with each of its several parts represented as resources. Within the DTS Collection endpoint structure, though, whatever entity is defined as a resource **cannot** have children. Any internal structure within that textual entity is represented only by way of the JSON properties of the resource: `dts:citeDepth`, `dts:citeStructure`, and any similar properties that are declared. Of course, the internal structure of a document may be accessed via the Document and Navigation endpoints.

## Using Collections to Represent Relationships

DTS does not specify any particular hierarchy of collections. A collection might provide all its child resources in a flat collection. Or a collection might group resources in sub-collections based on whatever relationships are deemed appropriate for the implementation at hand: geography, time, author, topic, etc. The collection structure can also be used to represent the relationships between parts or versions of a single work. A collection might, for example, link resources that are all chapters in one book. Or the collection might link transcriptions of different manuscripts attesting the same composition. Or, again, the collection might link competing transcriptions of a single manuscript. The collection itself simply represents some common relationship that links the resources or sub-collections that are its members. The precise nature of that relationship may be represented in the metadata for each collection. In this way, the collections endpoint provides for a highly flexible and adaptible representation of relationships.

For example, we might have a single book that is attested in several manuscripts and has also been translated into several languages. We might represent this situation by establishing a top-level collection representing the book. We might then create two collections as members for that top-level collection: one representing manuscripts and one representing translations. Each of those member collections would contain the relevant resources as its members:

**TODO: How should diagrams be included?**

                                               collection (book)
                                    --------------------------------------
                                    |                                     |
                    collection (manuscripts)                        collection (translations)
                    ------------------------                        -------------------------
                    |                      |                        |                       |
    resource (manuscript A)   resource (manuscript B)    resource (translation A)    resource (translation B)

Exactly how the metadata should express the nature of these relationships will be explored below.

Note that a resource or collection may have more than one parent. This allows the collection structure to be graph-like rather than a single branching tree. In a collection of letters, for example, we might want to organize the documents both by author and by the geographic location where it was composed. So these relationships might look something like this:

                                    collection (letters)
                        -------------------------------------------------
                        |                                               |

                collection (regions)                                collection (authors)
                --------------------                                ---------------------
                |                  |                                |                   |

    collection (Italy)     collection (Barbados)      collection (Medici)    collection (Smith)
    ------------------     ---------------------      -------------------    ------------------
            |                  |                               |                     |
            |                  ------resource(Smith from Barbados)--------------------
            |                                                  |
            ---------------resource (Medici from Italy)---------

The only limitation on this structure is that the top level must be a single collection with no parents. Unlike in an arbitrary graph, the relations between collections and resources is also hierarchical. There is a clear "top" to the structure, and the relationships are all hierarchical parent-child links. Peer or "sibling" relationships are simply inferred when entities share a common parent.

This does not mean, though, that the collections hierarchy must involve consistent "layers" or "levels." We might represent a collection of public statements made by an individual (Bill Nye) like this:

                                collection (statements by Bill Nye)
                ------------------------------------------------------------------------
                |                                                                       |
    collection (media formats)                                                     collection (2020)
    --------------------------------------------------------                       ------------------
    |                                                       |                             |  |
    collection (social media)                       collection (magazines)                |  |
    -------------------------                       ----------------------                |  |
    |                       |                               |                             |  |
    collection (twitter)    collection (facebook)           |                             |  |
        |                               |                   |                             |  |
        |                               |                   |                             |  |
        |                               resource (statement A) ----------------------------  |
        |                                                                                    |
        ------------------------------ resource (statement B) --------------------------------

Here we have a collection "statements by Bill Nye" with two sub-collections. The sub-collection "2020" holds all of the statements made in that year. A sibling collection at the same hierarchical level is labeled "media formats" and organizes documents based on the medium of their publication. The two resources "statement A" and "statement B" will ultimately be included in both collections. These resources are direct children of the "statements" collection. In the "media formats" collection, though, there are further levels of hierarchical organization before we get to those resources. "Media formats" has two further sub-collections, "social media" and "magazines". The "social media" collection contains its own child collections "twitter" and "facebook", but we have decided not to subdivide "magazines" any further. One statement by Nye, "statement A", was made in 2020. It was published first as a Facebook post and then re-published in a magazine article. So the resource entity representing the text of "statement A" is a direct child of three parent collections: "facebook", "magazines", and "2020". If we were concerned about absolute hierarchical levels, these parents would be at the fourth, third, and second levels of the hierarchy (respectively). The absolute hierarchical level of "statement A" would be conflicted. In the relatively free structuring constraints of the Collection endpoint, though, this is not necessarily a problem.

This example illustrates the flexibility of the Collection endpoint. We have chosen in this implementation not to assign any significance to absolute levels of the collections hierarchy. In another implementation we might decide that it is semantically useful to employ a more rigidly structured organization. You are free to use the basic relations of parent-child as you see fit. The only constraints that keep the Collection endpoint from being a truly arbitrary graph are (a) the requirement of a single top-level parent collection, and (b) the hierarchical, directional nature of the parent-child relationships.

## URLs for the Collection Endpoint

As with other endpoints, DTS does not require that you use a particular URL. If your implementation has a base URL `www.example.org/api` then you can call the endpoint "collection" and expose it at `www.example.org/api/collections`. Or you can choose to call it something else, perhaps "library", and expose the endpoint at `www.example.org/api/library`. In either case your documentation should clearly indicate the URL that corresponds in your implementation to the Collection endpoint.

**TODO: DTS does not specify URLs. Clients should discover URLs using navigation and link relations since URLs may differ among implementations.**

## Metadata Representation Scheme

### Metadata Vocabularies

Each collection or resource is a metadata record. Each record **must** take the form of Hydra-compliant JSON object. The DTS specification attempts to use existing standrd vocabularies wherever possible, supplemented by a small set of unique terms. The core set of attributes and properties available for a collection or resource is defined by three vocabuaries:

**Hydra Core Vocabulary**: This is a generic vocabulary developed to facilitate interoperable web APIs (https://www.hydra-cg.com/spec/latest/core/). These properties and attributes are used in DTS without any namespace prefix.

**DTS**: The DTS specification itself defines a small collection of terms (explained below) that define an item's relationships with other collections or resources. Properties from this vocabulary are given the prefix `dts`.

**TODO: DCMI?**
**DCMI Metadata Terms**
: DTS allows for the use of any terms from the Dublin Core Metadata Initiative's standard vocabulary for metadata (https://www.dublincore.org/specifications/dublin-core/). These terms are used in DTS with the prefix `dc`. Generally, any DCMI terms to be used are placed within the `dts:dublincore` property.

DTS is also designed to allow the expansion of these core vocabularies with other sets of terms as needed via the optional `dts:extensions` property discussed below.

These vocabularies must be declared once at the top level of any JSON object returned by the Collection endpoint. The global `@context` attribute must set the default vocabulary to Hydra Core Vocabulary and provide at least the DC and DTS namespace prefixes like this:

```json
  "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
  }
```

### Required Attributes and Properties

|      |                |
|----------|--------------------------|
|`title` | Each collection or resource assigned a single string as its human readable title.  Alternate titles or descriptions may be placed in `dts:dublincore` using the `dc:title` property, e.g. for internationalization.|
| `@id` | Each collection or resource is also assigned a unique identifier. This need not be human readable, but it will be the identifier used for this item in API requests to the Navigation and Document endpoints. It is strongly recommended that the `id` attribute be a valid URI. |
| `@type` | This attribute specifies whether the object is a "Collection" or "Resource". Only those two strings are valid in DTS as values for `@type`. |
| `totalItems` | The total number of items contained in the `member` property (irrespective of pagination). Depending on the direction in which the collection is traversed, these might be the current item's children or its parents. It will be identical to either `dts:totalChildren` or `dts:totalParents` depending on the direction of traversal.|
| `dts:totalChildren` | The total number of items contained in the `member` property if the request is pointed toward the item's children (`nav=children`).|
| `dts:totalParents` | The total number of items contained in the `member` property if the request is pointed toward the item's children (`nav=children`) |
| `dts:citeDepth` | **Required on Resource only**. This number provides the maximum depth of a readable resource's internal structural hierarchy (if any). In the case of a flat text, the `dts:citeDepth` value would be "1". If a resource represents a book divided into chapters, the `dts:citeDepth` would be "2". If those chapters are further subdivided into pages, it would be "3", etc. This number should correspond to the default citation structure used for requests via the Document endpoint.
| `dts:references` | **Required on Resource only**. This is the URI for the Navigation endpoint request corresponding to the current resource. **TODO: mandatory in children of `member`?**

### Optional Properties

|   |   |
|--------|---------------|
| `description` | In addition to the required `title` and `@id`, you may include a string that describes the collection or resource. Again, only one `description` is allowed per object, but additional descriptions may be placed in `dts:dublincore` as a `dc:description`, e.g. for internationalization. |
| `member` | By default `member` contains the children of a collection as an array of JSON objects. In this default use, a resource object would not include a `member` property, and neither would an empty collection. A request to the Collection endpoint can specify, though, that the members provide an array of the object's *parents* (`nav=parents`). In that case a resource object would also include a `member` property, since every resource will have at least one parent.|
| `dts:dublincore` | This property contains a single JSON object that can include any properties from the DCMI Metadata Terms along with the appropriate values. Each property must be prefixed with `dc`.|
| `dts:extensions` | Much like `dts:dublincore`, this property contains a single JSON object with attributes and/or properties from other vocabularies or ontologies. Each term should be preceded by a prefix identifying the vocabulary to which it belongs, and these prefixes should be declared in the top-level `@context` attribute alongside the `dts` and `dc` prefixes. |
| `dts:passage` | Where `dts:references` provides a link to the Navigation endpoing, `dts:passage` provides a direct link to the Document endpoint for fetching or (if supported) modifying the resource's textual representation. The `dts:references` property should be used to discover the possible routes to navigate into the text's internal structure. The `dts:passage` property, by contrast, should be used when requesting the whole textual representation at once. Since metadata for a textual resource may sometimes exist in a collection without the textual content being available, `dts:passage` is optional.|
| `dts:download` | If a downloadable form of the object (collection or resource) is available apart from text served via the Document endpoint, a URI (or array of URIs) may be included in `dts:download`. **TODO: This property should never provide alternate access to the same data accessible via a Document endpoint request.** |
| `dts:citeStructure` | The internal structure of a collection or resource can be provided as metadata here. It should take the form of a declared citation tree. **TODO: see [Sub-collection readable](#sub-collection-readable)**


## URI

### Query Parameters

The collections endpoint supports the following query parameters:

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | (Optional) The unique identifier for a collection or resource. This should be the same as the `@id` attribute for the requested item. If no `id` request parameter is supplied, the request should return the top-level collection for the server. |  GET, POST, PUT, DELETE |
| page | (Optional) The requested page of the current collection's members. **TODO: How is the number of pages discoverable?** |  GET    |
| nav  | (Optional) Determines the direction of traversal for a requested collection. By default DTS assumes a value of "children", so that the `member` property of the returned collection will contain its direct children. If a value of "parents" is supplied, the returned collection's `member` will contain the collection's parents instead. This value is passed on up or down the returned `member` hierarchy. So, in a request with "nav=parents", if the `member` parents themselves have higher level parent items, their nested `member` properties will also contain their parents rather than their children. | GET |
| token	 | (May be required by implementation) Authentication token for access control.	| GET, POST, PUT, DELETE |

### URI Template

Here is a template of the URI for the Collection API. The route itself (`www.example.com/api/collection/`) is up to the implementer.

```json
{
  "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
  },
  "@type": "IriTemplate",
  "template": "www.example.com/api/collection/?id={id}&page={page}&nav={nav}&token={token}",
  "variableRepresentation": "BasicRepresentation",
  "mapping": [
    {
      "@type": "IriTemplateMapping",
      "variable": "id",
      "required": true
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "page",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "nav",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "token",
      "required": false
    }

  ]
}
```

## Machine-Readable Endpoint Documentation

Machine-readable documentation for the endpoint should be made available in JSON-LD format following the Hydra Core Vocabulary specification. In this documentation the "supportedOperations" term must be used to specify which http methods the implementation supports for this endpoint. The value for "supportedOperations" should be an array of Hydra `Operation` objects, one object for each supported http method.

## GET Requests on the Collection Endpoint


### GET query parameters


No query parameters are required for a Collection `GET` request. If no `id` parameter is supplied, the request should return the top-level collection for the server. If no `page` parameter is supplied, and the implementation provides paginated results, the first page of results will be returned.

### GET request body

A `GET` request should not include a body. If one is included it should be ignored.


### GET Responses


#### Status codes

A successful `GET` request to the Collection endpoint should return the status code `200(OK)`.

If a `GET` request is unsuccessful because of problems with the request parameters, the return status code should be `400(Bad Request)` or a custom status code in the 4XX series signaling a more specific error.

If a `GET` request is unsuccessful because the specified `id` matches no collection or resource on the server, the return status code should be `404(Not Found)`

#### Successful GET response headers

The response after a successful `GET` request contains the following response headers:

| name | description |
|------|-------------|
| Link | The URL for the Hydra-compliant machine-readable `Collection` endpoint documentation with a "rel" value of "http://www.w3.org/ns/hydra/core#apiDocumentation |
| Content-Type | Content type of the response body (`application/ld+json`)|

#### Successful response body

The response body after a successful `GET` request should contain a JSON-LD object representing the requested collection or resource. This JSON should use the attributes and parameters outlined above under "[Metadata Representation Scheme](#metadata-representation-scheme)".

#### Unsuccessful response headers

An unsuccessful `GET` request should return the following headers:

| name | description |
|------|-------------|
| Link | The URL for the Hydra-compliant machine-readable `Collection` endpoint documentation with a "rel" value of "http://www.w3.org/ns/hydra/core#apiDocumentation |
| Content-type | Content type of the response body (`application/ld+json`) |

#### Unsuccessful response body

If a `GET` response returns an error code, the response body should contain a JSON-LD object following the [Hydra standard for error presentation](https://www.hydra-cg.com/spec/latest/core/#description-of-http-status-codes-and-errors).


For example, this would be a valid response body for a `400(Bad Request)` error:

```json
{
  "@context": "http://www.w3.org/ns/hydra/context.jsonld",
  "@type": "Status",
  "statusCode": 400,
  "title": "Invalid request parameters",
  "description": "The query parameters were not correct.",
}
```

For a `400(Bad Request)` error, the `description` should provide as much information as possible about which submitted data was missing or unacceptable. It should indicate whether:
- there were missing required query parameters
- any query parameters held unacceptable values, such as an id that does not match any collection or resource on the server
- the query parameters were acceptable but no page exists in the response data corresponding to the supplied `page` value

It is strongly recommended that projects implement more specific 4XX-series error responses to handle specific  errors. In such cases the `description` of the error response should include both an explanation of the issue and the URL for the necessary documentation.


### GET Example 1: Retrieving the root collection

In this example we request the top-level collection that groups texts into 3 categories. This collection has an `@id` attribute of "general". The three child collections have `@id` the attributes "cartulaires", "lasciva_roma", and "lettres_de_poilus".

**TODO: Note that these collection ids are not URIs**

#### GET request url :

The same result would be returned from either of these URLs:

- `www.example.com/api/collection/`
- `www.example.com/api/collection/?id=general`

Since we are requesting the top-level collection the `id` parameter is not strictly necessary.

#### Successful GET response status

- 200(OK)

#### Successful GET response headers

| Key | Value |
| --- | ----- |
| Link |  <www.example.com/api/collection/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation" |
| Content-Type | Content-Type: application/ld+json |

#### Successful GET response body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "general",
    "@type": "Collection",
    "totalItems": 2,
    "dts:totalParents": 0,
    "dts:totalChildren": 2,
    "title": "Collection Générale de l'École Nationale des Chartes",
    "dts:dublincore": {
        "dc:publisher": ["École Nationale des Chartes", "https://viaf.org/viaf/167874585"],
        "dc:title": [
            {"@language": "fr", "@value": "Collection Générale de l'École Nationale des Chartes"}
        ]
    },
    "member": [
        {
             "@id" : "cartulaires",
             "title" : "Cartulaires",
             "description": "Collection de cartulaires d'Île-de-France et de ses environs",
             "@type" : "Collection",
             "totalItems" : 10,
             "dts:totalParents": 1,
             "dts:totalChildren": 10
        },
        {
             "@id" : "lasciva_roma",
             "title" : "Lasciva Roma",
             "description": "Collection of primary sources of interest in the studies of Ancient World's sexuality",
             "@type" : "Collection",
             "totalItems" : 1,
             "dts:totalParents": 1,
             "dts:totalChildren": 1
        },
        {
             "@id" : "lettres_de_poilus",
             "title" : "Correspondance des poilus",
             "description": "Collection de lettres de poilus entre 1917 et 1918",
             "@type" : "Collection",
             "totalItems" : 10000,
             "dts:totalParents": 1,
             "dts:totalChildren": 10000
        }
    ]
}
```
**TODO: Woud we ever return a complete tree? I.e., are members ever nested?**

### GET Example 2: Child Collection Containing a Single Work

The example is a child of the parent root collection. It contains a single textual work as a member collection.

#### GET request url:

- `www.example.com/api/collection/?id=lasciva_roma`

#### Successful GET response status

- 200(OK)

#### Successful GET response headers

| Key | Value |
| --- | ----- |
| Link |  <www.example.com/api/collection/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation" |
| Content-Type | Content-Type: application/ld+json |


#### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "lasciva_roma",
    "@type": "Collection",
    "totalItems": 3,
    "dts:totalParents": 1,
    "dts:totalChildren": 3,
    "title" : "Lasciva Roma",
    "description": "Collection of primary sources of interest in the studies of Ancient World's sexuality",
    "dts:dublincore": {
        "dc:creator": [
            "Thibault Clérice", "http://orcid.org/0000-0003-1852-9204"
        ],
        "dc:title" : [
            {"@language": "la", "@value": "Lasciva Roma"},
        ],
        "dc:description": [
            {
                "@language": "en",
                "@value": "Collection of primary sources of interest in the studies of Ancient World's sexuality"
            }
        ],
    },
    "member": [
        {
            "@id" : "urn:cts:latinLit:phi1103.phi001",
            "title" : "Priapeia",
            "dts:dublincore": {
                "dc:type": [
                    "http://chs.harvard.edu/xmlns/cts#work"
                ],
                "dc:creator": [
                    {"@language": "en", "@value": "Anonymous"}
                ],
                "dc:language": ["la", "en"],
                "dc:description": [
                    { "@language": "en", "@value": "Anonymous lascivious Poems" }
                ],
            },
            "@type" : "Collection",
            "totalItems": 1,
            "dts:totalParents": 1,
            "dts:totalChildren": 1
        }
    ]
}
```

### GET Example 3: Child Collection Representing a Single Work

The example is a child collection. It represent a single textual work and its members are textual resources that are individual expressions of that work. These Resources are therefore Readable Collections.

**TODO: Readable collections?**


**TODO: Although, this is optional, the expansion of `@type:Resource`'s metadata is advised to avoid multiple API calls.**

#### GET request URL :

- `www.example.com/api/collection/?id=urn:cts:latinLit:phi1103.phi001`

#### Successful GET response status

- 200(OK)

#### Successful GET response headers

**TODO: include Link header to docs**
| Key | Value |
| --- | ----- |
| Link |  <www.example.com/api/collection/documentation>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation" |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
    },
    "@id": "urn:cts:latinLit:phi1103.phi001",
    "@type": "Collection",
    "title" : "Priapeia",
    "dts:dublincore": {
        "dc:type": ["http://chs.harvard.edu/xmlns/cts#work"],
        "dc:creator": [
            {"@language": "en", "@value": "Anonymous"}
        ],
        "dc:language": ["la", "en"],
        "dc:title": [{"@language": "la", "@value": "Priapeia"}],
        "dc:description": [{
           "@language": "en",
            "@value": "Anonymous lascivious Poems "
        }]
    },
    "totalItems" : 1,
    "dts:totalParents": 1,
    "dts:totalChildren": 1,
    "member": [
        {
            "@id" : "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
            "@type": "Resource",
            "title" : "Priapeia",
            "description": "Priapeia based on the edition of Aemilius Baehrens",
            "totalItems": 0,
            "dts:totalParents": 1,
            "dts:totalChildren": 0,
            "dts:dublincore": {
                "dc:title": [{"@language": "la", "@value": "Priapeia"}],
                "dc:description": [{
                   "@language": "en",
                   "@value": "Anonymous lascivious Poems "
                }],
                "dc:type": [
                    "http://chs.harvard.edu/xmlns/cts#edition",
                    "dc:Text"
                ],
                "dc:source": ["https://archive.org/details/poetaelatinimino12baeh2"],
                "dc:dateCopyrighted": 1879,
                "dc:creator": [
                    {"@language": "en", "@value": "Anonymous"}
                ],
                "dc:contributor": ["Aemilius Baehrens"],
                "dc:language": ["la", "en"]
            },
            "dts:passage": "www.example.com/api/document?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
            "dts:references": "www.example.com/api/navigation?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
            "dts:download": "https://raw.githubusercontent.com/lascivaroma/priapeia/master/data/phi1103/phi001/phi1103.phi001.lascivaroma-lat1.xml",
            "dts:citeDepth": 2,
            "dts:citeStructure": [
                {
                    "dts:citeType": "poem",
                    "dts:citeStructure": [
                        {
                            "dts:citeType": "line"
                        }
                    ]
                }
            ]
        }
    ]
}
```

### GET Example 4: Requesting a Resource

This example requests the metadata for a "resource", i.e. an item consisting of readable text. The response includes fields which identify the urls for the other two DTS api endpoints for further exploration of this resource: `dts:references` for retrieval of passage references via the Navigation endpoing, and `dts:passage` for retrieval of the entire collection of text passages (i.e the full document itself) via the Document endpoint.

#### GET request URL :

Note that there is nothing in the request URL to indicate that we are requesting a resource rather than a collection. This is indicated simply by the fact that the id value "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1" in fact identifies a resource on the server.

- `www.example.com/api/collection/?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1`

#### Successful Response Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response Example 1

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "@type" : "Resource",
    "title" : "Priapeia",
    "description": "Priapeia based on the edition of Aemilius Baehrens",
    "totalItems": 0,
    "dts:totalParents": 1,
    "dts:totalChildren": 0,
    "dts:dublincore": {
        "dc:title": [{"@language": "la", "@value": "Priapeia"}],
        "dc:description": [{
           "@language": "en",
            "@value": "Anonymous lascivious Poems "
        }],
        "dc:type": [
            "http://chs.harvard.edu/xmlns/cts#edition",
            "dc:Text"
        ],
        "dc:source": ["https://archive.org/details/poetaelatinimino12baeh2"],
        "dc:dateCopyrighted": 1879,
        "dc:creator": [
            {"@language": "en", "@value": "Anonymous"}
        ],
        "dc:contributor": ["Aemilius Baehrens"],
        "dc:language": ["la", "en"]
    },
    "dts:passage": "/api/dts/documents?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "dts:references": "/api/dts/navigation?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "dts:download": "https://raw.githubusercontent.com/lascivaroma/priapeia/master/data/phi1103/phi001/phi1103.phi001.lascivaroma-lat1.xml",
    "dts:citeDepth": 2,
    "dts:citeStructure": [
        {
            "dts:citeType": "poem",
            "dts:citeStructure": [
                {
                    "dts:citeType": "line"
                }
            ]
        }
    ]
}
```

#### Response Example 2

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#",
        "dc": "http://purl.org/dc/elements/1.1/"
    },
    "@id": "https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica",
    "@type" : "Resource",
    "title" : "Bucolica",
    "totalItems": 0,
    "dts:totalParents": 1,
    "dts:totalChildren": 0,
    "dts:passage": "/api/dts/documents?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica",
    "dts:references": "/api/dts/navigation?id=https://digitallatin.org/ids/Calpurnius_Siculus-Bucolica",
    "dts:download": "https://github.com/sjhuskey/Calpurnius_Siculus/blob/master/editio.xml",
    "dts:citeDepth": 2,
    "dts:citeStructure": [
        {
            "dts:citeType": "front"
        },
        {
            "dts:citeType": "poem",
            "dts:citeStructure": [
                {
                    "dts:citeType": "line"
                }
            ]
        }
    ]
}
```


### Paginated Child Collection

This is an example of a paginated request for a Child Collection's members.

#### Example of url :

- `/api/dts/collections/?id=lettres_de_poilus&page=19`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id" : "lettres_de_poilus",
    "@type" : "Collection",
    "totalItems" : 10000,
    "dts:totalParents": 1,
    "dts:totalChildren": 10000,
    "title": "Lettres de Poilus",
    "dts:dublincore": {
        "dc:publisher": ["École Nationale des Chartes", "https://viaf.org/viaf/167874585"],
        "dc:title": [
            {"@language": "fr", "@value" : "Lettres de Poilus"}
        ]
    },
    "member": ["member 190 up to 200"],
    "view": {
        "@id": "/api/dts/collections/?id=lettres_de_poilus&page=19",
        "@type": "PartialCollectionView",
        "first": "/api/dts/collections/?id=lettres_de_poilus&page=1",
        "previous": "/api/dts/collections/?id=lettres_de_poilus&page=18",
        "next": "/api/dts/collections/?id=lettres_de_poilus&page=20",
        "last": "/api/dts/collections/?id=lettres_de_poilus&page=500"
    }
}
```

### Parent Collection Query

This is an example of a query for the parents of a Collection. Note that, in this context, `totalItems` == `totalParents`

#### Example of url :

- `/api/dts/collections/?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1&nav=parents`

#### Headers

| Key | Value |
| --- | ----- |
| Content-Type | Content-Type: application/ld+json |

#### Response

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "@type" : "Resource",
    "title" : "Priapeia",
    "description": "Priapeia based on the edition of Aemilius Baehrens",
    "totalItems": 1,
    "dts:totalParents": 1,
    "dts:totalChildren": 0,
    "dts:dublincore": {
        "dc:title": [{"@language": "la", "@value": "Priapeia"}],
        "dc:description": [{
           "@language": "en",
            "@value": "Anonymous lascivious Poems "
        }],
        "dc:type": [
            "http://chs.harvard.edu/xmlns/cts#edition"
        ],
        "dc:source": ["https://archive.org/details/poetaelatinimino12baeh2"],
        "dc:dateCopyrighted": 1879,
        "dc:creator": [
            {"@language": "en", "@value": "Anonymous"}
        ],
        "dc:contributor": ["Aemilius Baehrens"],
        "dc:language": ["la", "en"]
    },
    "dts:passage": "/api/dts/documents?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "dts:references": "/api/dts/navigation?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "dts:download": "https://raw.githubusercontent.com/lascivaroma/priapeia/master/data/phi1103/phi001/phi1103.phi001.lascivaroma-lat1.xml",
    "dts:citeDepth": 2,
    "dts:citeStructure": [
        {
            "dts:citeType": "poem",
            "dts:citeStructure": [
                {
                    "dts:citeType": "line"
                }
            ]
        }
    ],
    "member": [
        {
            "@id" : "urn:cts:latinLit:phi1103.phi001",
            "title" : "Priapeia",
            "dts:dublincore": {
                "dc:type": [
                    "http://chs.harvard.edu/xmlns/cts#work"
                ],
                "dc:creator": [
                    {"@language": "en", "@value": "Anonymous"}
                ],
                "dc:language": ["la", "en"],
                "dc:description": [
                    { "@language": "en", "@value": "Anonymous lascivious Poems" }
                ],
            },
            "@type" : "Collection",
            "totalItems": 1,
            "dts:totalParents": 1,
            "dts:totalChildren": 1,
        }
    ]
}
```
