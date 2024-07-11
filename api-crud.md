# CRUD API Spec 1.0

### Definition
This **CRUD API Spec** describes a generic RESTful API specification to make the traditional entity operations:
- **C**reate
- **R**etrieve (single/multiple)
- **U**pdate (replace/patch)
- **D**elete (soft/hard)

### Objective
The objective of this specification is to aggregate common RESTful CRUD API patterns and other stablished specifications in order to achieve an universal CRUD API format. See [References](#References) for more details

### Inheritance

Except when explicitly overrided, all the "CRUD APIs" implemented in consonance with this specification:
- MUST follow the [HTTP Protocol](https://datatracker.ietf.org/doc/html/rfc2616)
- SHOULD follow the [REST Architecture Constraints and patterns](https://restfulapi.net/)

---

## 1. Vocabulary & terminology

The following keywords and terms are used in this document. They will have the meanings described below.

<details>
  <summary>Keywords</summary>

  ### Keywords
  The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
    NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and
    "OPTIONAL" in this document are to be interpreted as described in
    [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

</details>

<details>
  <summary>Additional terms and definitions</summary>

  ### Additional terms and definitions
  | Term | Definition |
  | --- | --- |
  | "Entity" or "Resource" | an unique entity within the API data domain |
  | "Resource ID" | an entity resource unique identification, such as specified in [#resource-ids] section |
  | "Entity Object" or "Document" | a single and unique object of an entity collection of objects |
  | "Entity Property" or "Object Property" or "Property" | an attribute of an entity |
  | `slug` | a string in format `^[a-zA-Z0-9_-]+$` |
  | `uuid-hex` | an UUID represented in [default encoding format](https://datatracker.ietf.org/doc/html/rfc4122#section-3) (hexadecimal, lowercase, hyphens) |
  | `uuid-base64` | an UUID byte array represented as a base64 string (without hyphens) |
  | `uuid-ejson` | an UUID represented as a JSON object and formatted as an EJSON Custom Type, as described in [ID section](#id) |
</details>

## 2. General specs

<details>
  <summary>Resource IDs</summary>

  ### Resource IDs
  - 🔴MUST be represented as a [`slug`](#1-vocabulary--terminology)
  - 🟡SHOULD be [`uuid-hex`](#1-vocabulary--terminology) or [`uuid-base64`](#1-vocabulary--terminology)

</details>

<details>
  <summary>JWTs</summary>

  ### JWTs
  When using JWTs, the following claims apply:

  - `sub` is 🔴REQUIRED to identify the end user performing the request. It 🟡SHOULD be an [URN](https://datatracker.ietf.org/doc/html/rfc8141) or an e-mail.
  - `email` 🟡SHOULD be used to help the API to set the author metadata information as an alternative to `sub` claim.
  - `azp` 🔵MAY indicate a Tenant ID that should be used to authorize and filter requests.

</details>

## 3. HTTP headers
<details>
  <summary>Request</summary>
  
  ### Request
  | Header | Requirements |
  | ---  | --- |
  | Accept | 🔴MUST be always present |

</details>

<details>
  <summary>Response</summary>
    
  ### Response

  | Header | Requirements |
  | ---  | --- |
  | Content-Type | 🔴MUST be always present and match the accepted media type. |
  | ETag | 🔴MUST be the same value as the [`_meta.hash`](#Versioning-and-hashing) property. |
  | Last-Modified | 🔴MUST be the same value as the [`_meta.updated.timestamp`](#Timestamps) property. |
  | Warning | 🔵MAY be used to indicate API warnings to the client, such as deprecation, quota, payment warnings, etc. |
  | X-Request-Id | 🟡SHOULD be a fresh [`uuid-hex`](#1-vocabulary--terminology). |

  #### Link Header
  - The `Link` header 🔴MUST be used in many cases described by this specification, like the [`Metadata section`](#8-metadata).
  - According to the HTTP Link Header spec, multiple links MUST be comma-separated. Example: `link1, link2`.
  - According to the HTTP Link Header spec, the relation (`rel`) attribute MUST be separated by a semicolon. Example: `link; rel=relation`.
  - All the URIs in Link header 🟡SHOULD be relative paths when the links are relative, in order to reduce the HTTP payload length.

  <details>
    <summary>All the link relations and spec references</summary>
    
  | Relation | Usage | Reference | Example |
  | --- | --- | --- | --- |
  | about | The URL that should be called to include metadata information with the request. | [Metadata - General specification](#General-specification) | `/users/ID?meta=true; rel=about` |
  | author | The author ID that performed the last modification to the entity object. | [Metadata - `_meta.updated.author`](#Timestamps) | `urn:users:ID; rel=author` |
  | collection | The URL that points to the entity list when dealing with single objects. | --- | `/users; rel=collection` |
  | convertedfrom | Indicates the original resource the entity object was cloned from. | --- | `/people/ID; rel=convertedfrom` |
  | current | The current page in a list of resources. | --- | `/users?page=2&page_size=20; rel=current` |
  | describedby | The entity schema URL. | --- | `https://myapi.com/schemas/users.json; rel=describedby` |
  | edit | The preferred endpoint to edit this entity object. | --- | `/users/ID; rel=edit; method=patch` |
  | first | The first page in a list of resources. | --- | `/users?page=1&page_size=20; rel=first` |
  | last | The last page in a list of resources. | --- | `/users?page=5&page_size=20; rel=last` |
  | next | The next page in a list of resources. | --- | `/users?page=3&page_size=20; rel=next` |
  | prev | The previous page in a list of resources. | --- | `/users?page=1&page_size=20; rel=prev` |
  | service-desc | The OpenAPI Spec URL of the current API. | --- | `https://myapi.com/docs/oas.json; rel=service-desc` |
  | service-doc | The Swagger UI URL of the current API. | --- | `https://myapi.com/docs/swagger; rel=service-doc` |
    
  </details>
</details>

## 4. URL parameters
<details>
  <summary>Expand</summary>

  ### Path parameters
  | Parameter | Requirements |
  | ---  | --- |
  | `:id` | 🔴MUST be `alphanumeric+hiphen+under`. 🟡SHOULD be a valid UUID encoded as hex or base64 string |
  | `:entity` | 🔴MUST be `alphanumeric+hiphen+under`. |
  
</details>

## 4. Body payloads
<details>
  <summary>No envelope usage</summary>

  ### No envelope usage
  - Request and response bodies 🔴MUST represent literally the entities objects and its top-level attributes
  - Entities objects 🔴MUST NOT be wrapped in envelopes, such as `data`, `response`, `result` or any similar.
  - The API 🔴MUST NOT respond with any virtual property (a property that is not stored within the entity object) to reference any entity links.
  - Related entity resources that are not explicitly stored inside entity objects 🔴MUST be referenced using the [Link header](#Link-Header) and these relations 🔴MUST use one of the [IANA Link Relations](https://www.iana.org/assignments/link-relations/link-relations.xhtml).

</details>

<details>
  <summary>JSON payloads</summary>
  
  ### JSON payloads
  - When using JSON as the serialization method for request/response payloads, the content-type 🟡SHOULD be `application/vnd.ejson+json`.
  - When serializing language-specific types, EJSON serialization 🟡SHOULD be used in order to preserve the type information and to make sure that strongly-typed values are always represented as strings
    - The default [MongoDB EJSON specification in relaxed format](https://www.mongodb.com/docs/upcoming/reference/mongodb-extended-json/) 🔴MUST be used to serialize the most common complex types
    - Additionally, the [Meteor EJSON specification](https://docs.meteor.com/api/ejson#EJSON-addType) 🔵MAY be used to serialize other custom types
  
</details>

<details>
  <summary>Error payloads</summary>
  
  ### Error payloads
  - Unsuccessful HTTP Requests 🔴MUST respond with the appropriate HTTP Status Codes and a payload with additional error details.
  - The error payload 🟡SHOULD have at least:
    1. an error unique code, such as a number or a string;
    2. a human message describing the error.
  - The error payload 🔵MAY also have additional properties to help developers and systems to deal with the error, like debug and documentation URLs, tracing information, etc.
  
</details>

## 5. Type definitions
<details>
  <summary>ID</summary>

  ### ID
  - The ID type 🔴MUST be an object with the following properties below:
    - `$type`: must be `"uuid"`
    - `$hex`: the standardized `uuid-hex` representation of the UUID value
    - `$64`: the alternative `uuid-base64` representation of the UUID value
  - This specification is agnostic to the specific language implementation of the UUID type, whether if it is a buffer, string or other binary format.

  **Example**
  ```json
  {
    "$type": "uuid",
    "$hex": "0190a295-e942-75fd-8495-894efaf93a78",
    "$64": "AZCilelCdf2ElQAAiU76+Q",
  }
  ```
 
</details>

## 6. Entity anatomy
<details>
  <summary>Property keys conventions</summary>

  ### Property keys conventions
  - Properties prefixed by underscores `_` 🟡SHOULD NOT be used unless specified by this specification. They are reservated for properties that affects the operations behaviors.
  - It is 🟡RECOMMENDED, although it is not important, to use `camelCase` as the default naming convention for properties keys 
 
</details>

<details>
  <summary>_id</summary>

  ### `_id`
  - All entity objects 🔴MUST have an `_id` property.
  - The type of this property 🟡SHOULD be the [ID type](#id)
    - When it is not possible to use the [ID type](#id), the property type 🟡SHOULD be a [`slug`](#1-vocabulary--terminology)
 
</details>

<details>
  <summary>_meta</summary>

  ### `_meta`
  - All entity objects 🔴MUST have a `_meta` property.
  - Its type, values and other specifications 🔴MUST follow the [Metadata section](8-Metadata).
 
</details>

<details>
  <summary>_tid</summary>

  ### `_tid`
  - The `_tid` property 🔵MAY be used to indicate a Tenant ID that the Document is subject to.
  - Its type 🟡SHOULD be the [ID type](#id)
 
</details>

## 7. Endpoints

### Create one

```
POST /:entity/
```

<details>
  <summary>Request body</summary>

  -  Top level: 🔴MUST be an entity object
  -  The `_id` property is accepted 🔵MAY be sent if it matches the adequate entity ID type
    
</details>

### Get one
```
GET /:entity/:id
```

<details>
  <summary>Response headers</summary>

  - Link rel=collection header

</details>

<details>
  <summary>Response body</summary>

  - The endpoint 🔴MUST return the integral entity object without any modification.

</details>

### Patch one by id

```
PATCH /:entity/:id
```

<details>
  <summary>Request body</summary>

  - Top level: 🔴MUST be either one of:
    - (Short form) a partial entity object that describes each property/value that must be replaced
    - (Long form) a [JSON Patch](https://jsonpatch.com/) operations array
  - `_id` is 🔴FORBIDDEN

</details>

<details>
  <summary>Response body</summary>

  - The endpoint 🟡SHOULD return the entity object after the performed modifications.
    - Otherwise, the endpoint 🔴MUST return an empty body.

</details>

### Replace one by id

```
PUT /:entity/:id
```

<details>
  <summary>Request body</summary>

  - The same definition as [Create one](#create-one)

</details>

<details>
  <summary>Response body</summary>

  - The endpoint 🟡SHOULD return the entity object after the performed replacement.
    - Otherwise, the endpoint 🔴MUST return an empty body.

</details>

### Patch multiple by filter

```
PUT /:entity/:id?filter=[filter]
```

<details>
  <summary>URL Parameters</summary>

  - The query string 🔴MUST have a `filter` parameter with the appropriate query filter, regardless its syntax

</details>

<details>
  <summary>Request body</summary>

  - The same definition as [Patch one by id](#Patch-one-by-id)

</details>

<details>
  <summary>Response body</summary>

  - The endpoint 🟡SHOULD return the integral entity objects after the performed modifications.
    - Otherwise, the endpoint 🔴MUST return an empty body.

</details>

### Delete one by id

```
DELETE /:entity/:id?[force=true]
```
 
<details>
  <summary>URL Parameters</summary>

  - The query string 🟡SHOULD accept a `force=true` parameter if the CRUD API has the capability of soft delete.
    - If the parameter is present, the resouce 🔴MUST be permanently deleted.
    - Otherwise, the entity object should be softly deleted in the server by setting the property `_meta.status` to `archived`, as described in section XXXXX.

</details>

<details>
  <summary>Response body</summary>

  - The endpoint 🔴MUST return an empty body.

</details>

<details>
  <summary>List many</summary>

  ### List many
  ```
  GET /:entity?[status]&[sort]&[fields]&[page]&[per_page]
  ```
  - Request query
    - `status` (optional):
      - one of the enum values: `published` (default if ommited), `drafts` or `archived`.
      - Multiple values are accepted in comma-separated format.
      - Example: `published,drafts`.
    - `sort` (optional):
      - one or many entity property keys for sorting.
      - Multiple values are accepted in comma-separated format.
      - Dot-notation should be used for nested properties.
      - The `-` hyphen should be used together with a property key to indicate a descending order.
    - `fields` (optional):
      - one or many entity property keys for projection.
      - Multiple values are accepted in comma-separated format.
      - Dot-notation should be used for nested properties.
      - The `-` hyphen should be used together with a property key to omit a property.
  
  #### Pagination

  Link rels
  first
  current
  last
  next
  prev
</details>

## 8. Metadata
<details>
  <summary>General specification</summary>

  ### General specification
  - All entities compliant with this specification 🔴MUST include the metadata information described in this section.
  - Single entities responses 🔴MUST use the appropriate HTTP headers defined in this section to provide metadata information.
  - Entities responses containing either a single or multiple objects, 🟡SHOULD include metadata information aggregated inside the `_meta` property.
    - When the `?meta=true` parameter is included in the endpoint URL query, the objects 🔴MUST include the `_meta` property.
  - An entity object 🔵MAY have duplicated metadata information in other entity properties beyond the `_meta` property.
  - Although many API data sources do not have the capability of storing nested objects, thie `_meta` property 🔴MUST be always a nested object regardless the storage implementation.
  - The `_meta` property 🔴MUST be ignored in every write request payload (create, patch, replace). The metadata information is intended to be set by the API server, not the client.
</details>


<details>
  <summary>Versioning and hashing</summary>

  ### Versioning and hashing

  | Header | Value |
  | --- | --- |
  | ETag | `_meta.hash` |
  
  - All entities 🔴MUST include a `_meta.version` property. This property value 🔴MUST start with the number 1 at the object creation and 🔴MUST be incremented by one at every modification of the object (patch or replace operations).
  - All entities 🔴MUST have a `_meta.hash` property. The value 🔴MUST be a hashed string of the concatenation of the shortest value of the `_id` property and the `_meta.version` value (i.e. `[_id.$base64][_meta.version]`. The hashing algorithm 🟡SHOULD be CRC32 for optimization.

</details>

<details>
  <summary>Timestamps</summary>

  ### Timestamps

  | Header | Value |
  | --- | --- |
  | Last-Modified | `_meta.updated.timestamp` 🔴REQUIRED |
  | Link | `urn:<_meta.updated.author>; rel=author` 🔵OPTIONAL |

  - All entities 🔴MUST include a `_meta.events` property with the following properties:
    - `created`: (🔴REQUIRED) metadata information about the creation event
    - `updated`: (🔴REQUIRED metadata information about the last time the object was modified, including the creation
  - The `created` and `updated` properties 🔴MUST be objects with the following properties:
    - `timestamp` (🔴REQUIRED): a JavaScript date representing when the event occurred
    - `author` (🔵OPTIONAL): the author ID or e-mail of the operation actor, if applicable
    - `reason` (🔵OPTIONAL): the system comments about the operation
    - `comments` (🔵OPTIONAL): the end-user comments about the operation

</details>

<details>
  <summary>Origin</summary>

  ### Origin

  | Header | Value |
  | --- | --- |
  | Link | `<URL>; rel=convertedfrom` 🔵OPTIONAL if it is possible to link using an URL, either absolute or relative |

  - If an entity object was cloned/generated from another object, it 🟡SHOULD have a `_meta.origin` property with the following nested properties:
    - `realm`: 🔴MUST be used when the object was cloned/generated from another source. Its value 🟡SHOULD be an universal representation of the source, like an Internet Domain Name.
    - `type`: 🔴MUST be used when the source object had a different type than the current object. Its value 🟡SHOULD be the name of the foreign entity.
    - `id`: 🔴MUST be used to indicate the original Resource ID.

</details>

<details>
  <summary>Lifecycle</summary>

  ### Lifecycle

  Lifecycle is intended to logically express two capabilities of a CRUD API:
  1. Soft-delete
  2. Data staging

  #### The status property
  - Regardless the domain-specific lifecycle status of an entity, the `_meta.status` 🔴MUST NOT be used in any case different than the cases below and any other `_meta.status` values 🔴MUST be ignored by the CRUD API.
  - Documents without the `_meta.status` property 🟡SHOULD be considered as "published", "mature" and "integral" versions of the object.

  #### Soft delete
  - Some CRUD APIs are capable of soft-deleting entity objects. This consists in put objects into an "archived" state and separate them off the rest of the entity objects, in order to retain the information for a certain period of time.
  - Regardless if the "archived" objects are stored within the same "unarchived" objects storage, it is a best pratice to mark the objects as archived anyway.
  - To achieve this, a CRUD API with the soft-delete capability 🔴MUST mark the archived objects with the `_meta.status` value set to `"archived"`, as also described in the [DELETE] endpoint.
  - Documents marked with `_meta.status = "archive"` 🟡SHOULD NOT be retrieved in read operations unlesss explicitly specified by the client. 

  #### Data staging
  - Some CRUD APIs are capable of "stage" entity objects until they are in a certain maturity stage. Some of these stagings could be drafts of a blog post or even SAGA Patterns applied to entities. Data staging could also be used to perform transactions in copies ("branched objects") before apply modifications to an object.
  - To achieve this, a CRUD API with data staging capability 🔴MUST mark the staging/draft objects with the `_meta.status` value set to `"draft"`.
  - Documents marked with `_meta.status = "draft"` 🟡SHOULD NOT be retrieved in read operations unlesss explicitly specified by the client.

</details>

---

## References

### Standards
- HTTP Patch: [RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789)
- UUID v7: [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562#section-5.7)
- JSON Web Tokens (JWT) Claims: [RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519) | [IANA Claims Registry](https://www.iana.org/assignments/jwt/jwt.xhtml)
- Web Linking: [RFC 8288#3](https://httpwg.org/specs/rfc8288.html#header) | [IANA HTTP Fields Registry](https://www.iana.org/assignments/http-fields/http-fields.xhtml)
- JSON Patch: [Website](https://jsonpatch.com/) | [RFC 6901](https://datatracker.ietf.org/doc/html/rfc6901/)
- EJSON: [EJSON Relaxed Format](https://www.mongodb.com/docs/upcoming/reference/mongodb-extended-json/) | [EJSON Custom Type](https://docs.meteor.com/api/ejson#EJSON-addType)

### Inspirations
- Salesforce REST API: [Docs]() | [Warning Header](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/headers_warning.htm) | [REST Endpoint Anatomy](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_rest_resources.htm)
- Github REST API: [Docs](https://docs.github.com/pt/rest) | [Pagination](https://docs.github.com/pt/rest/using-the-rest-api/using-pagination-in-the-rest-api?apiVersion=2022-11-28)
- Notion API: [Docs](https://developers.notion.com/reference) | [Pagination](https://developers.notion.com/reference/intro#parameters-for-paginated-requests)
- Wordpress REST API: [Docs](https://developer.wordpress.org/rest-api/)

## License

> This specification Source Code is subject to the terms of the Mozilla Public License, v. 2.0. If a copy of the MPL was not distributed with this file, you can obtain one at http://mozilla.org/MPL/2.0/.
