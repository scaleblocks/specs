# CRUD API Spec 1.0

### Definition
This **CRUD API Spec** describes a generic RESTful API specification to make the traditional entity operations:
- **C**reate
- **R**etrieve (single/multiple)
- **U**pdate (replace/patch)
- **D**elete (soft/hard)

### Objective
The objective of this specification is to aggregate common RESTful CRUD API patterns and other stablished specifications in order to achieve an universal CRUD API format.

### Inheritance
Except when explicitly overrided, all the "CRUD APIs" implemented inconsonance with this specification:
- MUST follow the [HTTP/1.1 standard](https://datatracker.ietf.org/doc/html/rfc2616)
- SHOULD follow the [REST Architecture Constraints and patterns](https://restfulapi.net/)

[JWTs] | [Required properties] | [Metadata]

---

## 1. Vocabulary & terminology
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
  | `slug` | a string in format `^[a-zA-Z0-9_-]+$` |
  | `uuid-hex` | an UUID represented in [default encoding format](https://datatracker.ietf.org/doc/html/rfc4122#section-3) (hexadecimal, lowercase, hyphens) |
  | `uuid-base64` | an UUID byte array represented as a base64 string (without hyphens) |
  | `uuid-ejson` | an UUID represented as a JSON object and formatted as an EJSON Custom Type, as described in [ID section](#id) |
  | Resource ID | an entity resource unique identification, such as specified in [#resource-ids] section

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
  | Content-Type | 🔴MUST be always present and match the accepted media type |
  | X-Request-Id | 🟡SHOULD be an [`uuid-hex`](#1-vocabulary--terminology) |
  | ETag | 🟡SHOULD be the same value as the `_meta.hash` property |
  | Link | Related entity resources, regardless if they were stored within the entity object or not, 🟡SHOULD be referenced using the [HTTP Link header](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Headers/Link) and these relations 🔴MUST use one of the [IANA Link Relations](https://www.iana.org/assignments/link-relations/link-relations.xhtml). |

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
  - Related entity resources that are not explicitly stored inside entity objects 🔴MUST be referenced using the [HTTP Link header](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Headers/Link) and these relations 🔴MUST use one of the [IANA Link Relations](https://www.iana.org/assignments/link-relations/link-relations.xhtml).

</details>

<details>
  <summary>JSON payloads</summary>
  
  ### JSON payloads
  - When using JSON as the serialization method for request/response payloads, the content-type 🟡SHOULD be `application/vnd.ejson+json`.
  - When serializing language-specific types, EJSON serialization 🟡SHOULD be used in order to preserve the type information and to make sure that strongly-typed values are always represented as strings
    - The default [MongoDB EJSON specification in relaxed format](https://www.mongodb.com/docs/upcoming/reference/mongodb-extended-json/) 🔴MUST be used to serialize the most common complex types
    - Additionally, the [Meteor EJSON specification](https://docs.meteor.com/api/ejson#EJSON-addType) 🔵MAY be used to serialize other custom types
  
</details>

## 5. Type definitions
<details>
  <summary>ID</summary>

  ### ID
  - The ID type 🔴MUST be an object with the following properties below:
    - `$type`: must be `"uuid"`
    - `$hex`: the standardized `uuid-hex` representation of the UUID value
    - `$64`: the alternative `uuid-base64` representation of the UUID value
 
</details>

## 6. Entity anatomy
<details>
  <summary>Property keys conventions</summary>

  ### Property keys conventions
  - Properties prefixed by underscores `_` 🟡SHOULD NOT be used unless specified by this specification
  - It is 🟡RECOMMENDED, although it is not important, to use `camelCase` as the default naming convention for properties keys
 
</details>

<details>
  <summary>_id</summary>

  ### `_id`
  - All entity objects 🔴MUST have an `_id` property.
  - The type of this property 🟡SHOULD be the [ID type](#id)
    - When it is not possible to use the [ID type](#id), the property type 🟡SHOULD be a [`slug`](#1-vocabulary--terminology)
 
</details>

## 7. Endpoints
<details>
  <summary>Create one</summary>

  ### Create one
  ```
  POST /:entity/
  ```
  - Request body
    -  Top level: an entity object
    -  `_id` property is accepted if is a valid UUID-ejson
</details>

<details>
  <summary>Get one</summary>

  ### Get one
  ```
  GET /:entity/:id
  ```
</details>

<details>
  <summary>Patch one by id</summary>

  ### Patch one by id
  ```
  PATCH /:entity/:id
  ```
  - Request body
    - Top level: either one of:
      - (Short form) a partial entity object that describes each property/value that must be replaced
      - (Long form) a [JSON Patch](https://jsonpatch.com/) operations array
    -  `_id` is forbidden
</details>

<details>
  <summary>Replace one by id</summary>

  ### Replace one by id
  ```
  PUT /:entity/:id
  ```
  - The same definition as [Create one](#create-one)

</details>

<details>
  <summary>Patch one by id</summary>

  ### Replace one by id
  ```
  PUT /:entity/:id
  ```
  - The same definition as [Create one](#create-one)

</details>
 
<details>
  <summary>Delete one by id</summary>

  ### Replace one by id
  ```
  DELETE /:entity/:id?[force=true]
  ```
  - Request query
    - `force=true` (optional): if it is present, the resouce MUST be permanently deleted. Otherwise, the entity object should be softly deleted in the server by setting the property `_meta.status` to `ARCHIVED`.
  - Response status: `204`

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

</details>

## 8. Metadata
<details>
  <summary>Create one</summary>

  ### Create one
  ```
  POST /:entity/
  ```
  - Request body
    -  Top level: an entity object
    -  `_id` property is accepted if is a valid UUID-ejson
</details>

---

## References

- HTTP Patch: [RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789)
- UUID v7: [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562#section-5.7)
- JSON Patch: [Website](https://jsonpatch.com/) | [RFC 6901](https://datatracker.ietf.org/doc/html/rfc6901/)
- EJSON: [EJSON Relaxed Format](https://www.mongodb.com/docs/upcoming/reference/mongodb-extended-json/) | [EJSON Custom Type](https://docs.meteor.com/api/ejson#EJSON-addType)

## License

> This specification Source Code is subject to the terms of the Mozilla Public License, v. 2.0. If a copy of the MPL was not distributed with this file, you can obtain one at http://mozilla.org/MPL/2.0/.
