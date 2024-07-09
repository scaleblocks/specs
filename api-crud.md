# CRUD API Spec

[Vocabulary] | [JWTs] | [Required properties] | [Metadata] 

## HTTP Headers

- Request
  - `Accept` must be always present
- Response
  - `Content-Type` must be `application/vnd.ejson+json` when the response body is JSON
  - `X-Request-Id`: should be an UUID v7 hex string
  - `ETag`: should be the same value as the `_meta.hash` property

## Parameters

- `:id` must be a valid UUID encoded as hex or base64 string.
- `:entity` must be `alphanumeric+hiphen+under`

## Endpoints

Endpoints are the paths+verbs pairs provided by a CRUD API.

| Verb | Path | Operation |
| --- | --- | --- |
| `POST` | `/:entity/` | Create one |

#### Create one
```
POST /:entity/
```
- Request body
  -  Top level: an entity object
  -  `_id` property is accepted if is a valid UUID-ejson
 
#### Get one
```
GET /:entity/:id
```

#### Patch one by id
```
PATCH /:entity/:id
```
- Request body
  - Top level: either one of:
    - (Short form) a partial entity object that describes each property/value that must be replaced
    - (Long form) a [JSON Patch](https://jsonpatch.com/) operations array
  -  `_id` is forbidden
 
#### Replace one by id
```
PUT /:entity/:id
```
- The same definition as [Create one](#create-one)
 
#### Replace one by id
```
DELETE /:entity/:id?[force=true]
```
- Request query
  - `force=true` (optional): if it is present, the resouce MUST be permanently deleted. Otherwise, the entity object should be softly deleted in the server by setting the property `_meta.status` to `ARCHIVED`.
- Response status: `204`

#### List
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
   
## JWTs

When using JWTs, the following claims apply:

- `sub` is required to identify the end user performing the request. It should be an ID or an e-mail.
- `email` (optional) should be used to help the API to set the author metadata information as an alternative to `sub` claim.
- `azp` (optional) may be indicate a Tenant ID that should be used to authorize and filter requests.

## Vocabulary

- `alphanumeric+hiphen+under` format describes a string in format `<ALPHA | DIGIT | “-” | “_”>` (RFC 822#3.3) and case-insensitive (RFC 1035#2.3.3)
- `uuid-ejson` format describes an UUID represented as a JSON object and conventioned as an EJSON Custom Type, with the following properties:
  - `$type`: `uuid`
  - `$hex`: the original hexadecimal string representation of the UUID, separated by hyphens
  - `$64`: a base-64 string representation of the UUID value

## Specifications and patterns

- HTTP Patch: [RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789)
- UUID v7: [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562#section-5.7)
- JSON Patch: [Website](https://jsonpatch.com/) | [RFC 6901](https://datatracker.ietf.org/doc/html/rfc6901/)
- EJSON: [EJSON Relaxed Format](https://www.mongodb.com/docs/upcoming/reference/mongodb-extended-json/) | [EJSON Custom Type](https://docs.meteor.com/api/ejson#EJSON-addType)

## License

> This Source Code Form is subject to the terms of the Mozilla Public
  License, v. 2.0. If a copy of the MPL was not distributed with this
  file, You can obtain one at http://mozilla.org/MPL/2.0/.
