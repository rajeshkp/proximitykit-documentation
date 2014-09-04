---
layout: proximitykit
---

# Proximity Kit Admin API

This document describes the API designed to integrate Proximity Kit with other systems. It is specifically intended to manage creation and deletion of kits, regions, the associated meta data.

Data synchronization, communicating with the SDKs, configuring hardware is not part of the Admin API. Radius Networks has [other resources](http://developer.radiusnetworks.com/) for many of those needs.

## Overview

The Proximity Kit API follows the [JSON API](http://jsonapi.org/format/) format.

JSON API uses IDs for linkage, which makes it possible to cache documents from compound responses and then limit subsequent requests to only the documents that aren't already present locally.

Resource relationships are created through the use of URI templates and resource identifiers. To prevent future breakage we recommend creating the URIs from the templates over hardcoding to the individual resources.

## Headers

### Authorization

The API Key is passed via the Authorization header:

    Authorization: Token token="secret"

The API Key is associated with your account and has access to all the resources associated with your account. Account-specific API keys have different permissions than the web login users that can interact with the dashboard, and the access may be different.

Example:

    curl -H 'Authorization: Token token="sandbox"' "http://proximitykit.com/api/v1.json"

If you do not have an API key, [please contact us](mailto:support@radiusnetworks.com?subject=Proximity Kit API Token Request) for one.

Note: Per [RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec2.html#sec2.2) the Authorization Header's token needs to be surrounded by double quotes (`"`).

### Content Type

The content type is `vnd+json`, and should be set in the Content-Type header:

    Content-Type: application/vnd.api+json

# API Root

    GET /api/v1

The list of links for the resources available to your account.

Root Document:

`GET /api/v1`

```json
{
  "version": "1.0",
  "links": {
    "users.kits": "http://proximitykit.com/api/v1/kits/{users.kits}"
  },
  "users": [
    {
      "id": 3,
      "links": {
        "kits": [52,53]
      }
    }
  ]
}
```

Key                  | Value
------------------   |---------------------------------------
`version`            | The media type version
`links[users.kits]`  | The url template for a kit resource
`users[links][kits]` | List of kit IDs that belong to a user


Curl Example:

```
% curl -s \
       -X GET \
       -H 'Authorization: Token token="secret"' \
       -H "Content-Type: application/vnd.api+json" \
       http://proximitykit.com/api/v1
```

# Kits

## Kit Resource

`GET /api/v1/kits/:id`

Return the kit resource, describing the attributes and related beacons.

```json
{
  "version": "1.0",
  "links": {
    "kits.beacons": "http://proximitykit.com/api/v1/beacons/{kits.beacons}"
  },
  "kits": [
    {
      "name": "Ancient Rome",
      "id": 52,
      "links": {
        "beacons": [2,3,4,5]
      }
    }
  ]
}
```

Key                     | Value
----------------------- | ------------------------------------------
`version`               | The media type version
`links[kits.beacons]`   | The url template for an beacon resource
`kits[links][beacons]`  | List of kit IDs that belong to a user


Curl Example:

```
% curl -s \
       -X GET \
       -H 'Authorization: Token token="secret"' \
       -H "Content-Type: application/vnd.api+json" \
       http://proximitykit.com/api/v1/kits/52
```

## Listing Kits

`GET /api/v1/kits/:id`

Return a paginated of kit resources.

```json
{
  "version": "1.0",
  "meta": {
    "current": "http://proximitykit.com/api/v1/kits?page=3",
    "next": "http://proximitykit.com/api/v1/kits?page=4",
    "previous": "http://proximitykit.com/api/v1/kits?page=2",
    "first": "http://proximitykit.com/api/v1/kits?page=1",
    "last": "http://proximitykit.com/api/v1/kits?page=10"
  },
  "links": {
    "kits.beacons": "http://proximitykit.com/api/v1/beacons/{kits.beacons}"
  },
  "kits": [
    {
      "name": "Ancient Rome",
      "id": 52,
      "links": {
        "beacons": [ 2, 3, 4, 5 ]
      }
    },
    //...
  ]
}
```

Key                     | Value
----------------------- | ------------------------------------------
`version`               | The media type version
`links[kits.beacons]`   | The url template for an beacon resource
`kits[links][beacons]`  | List of kit IDs that belong to a user
`meta[next]`            | URI of the next page of results
`meta[previous]`        | URI of the previous page of results
`meta[first]`           | URI of the first page of results
`meta[last]`            | URI of the last page of results



## Creating a Kit

To create a kit make a post request to the `/api/v1/kits`

    POST /api/v1/kits

The response will include the JSON representation of the newly created kit as well as a `Location` header with the URI of the resource.

Example:

```json
{"kits": [{ "name": "My New Name" }] }
```


Required Parameters:


Parameter             | Description
----------------------|------------------------------------------
name                  | The name of the kit


Curl Example:

```
% curl -sv \
       -X POST \
       -H 'Authorization: Token token="secret"' \
       -H "Content-Type: application/vnd.api+json" \
       -d ' {"kits":[{ "name": "My Name" }]}'
       http://proximitykit.com/api/v1/kits
< HTTP/1.1 201 Created
< Location: http://proximitykit.com/kits/56
< Content-Type: application/vnd.api+json; charset=utf-8
{
  "kits": [
    {
      "name": "My Name",
      "id": 56,
      "links": { "beacons": [] }
    }
  ]
}
```

## Updating a Kit

To update a kit make a put request to the `/api/v1/kits`.

`PUT /api/v1/kit/:id`

Example:

```json
{"kits": [{ "name": "My New Name" }] }
```

Parameters:

Parameter             | Description
----------------------|------------------------------------------
`name`                | The name of the kit

Curl Example:

```
% curl -sv \
       -X PUT \
       -H 'Authorization: Token token="secret"' \
       -H "Content-Type: application/vnd.api+json" \
       -d ' {"kits":[{ "name": "Silly Name" }]}' \
       http://proximitykit.com/api/v1/kits/56
< HTTP/1.1 204 No Content
```

## Deleting a Kit

To delete a kit make a delete request to the individual kit url.

```
DELETE /api/v1/kit/:id
```

The response be `201 No Content` if the delete was successful.

Curl Example:

```
% curl -sv \
       -X DELETE \
       -H 'Authorization: Token token="secret"' \
       -H "Content-Type: application/vnd.api+json"
       http://proximitykit.com/api/v1/kits/56
< HTTP/1.1 204 No Content
```

# Beacons

## Beacon Resource

`GET /api/v1/beacons/:id`

This will return an beacon resource.

```
{
  "links": {
    "beacons.attributes": "http://proximitykit.com/api/v1/beacon_attributes/{beacons.attributes}"
  },
  "beacons": [
    {
      "id": 6,
      "name": "Earth",
      "uuid": "E58171D9-8398-4453-A991-5593682DDB56",
      "major": 1000,
      "minor": 1,
      "links": {
        "attributes": [3,4]
      }
    }
  ]
}
```

Key                           | Value
----------------------------- | ---------------------------------------------------
`version`                     | The media type version
`links[beacons.attributes]`   | The url template for a beacon attribute resource
`beacons[name]`               | Display name for the beacon
`beacons[uuid]`               | Orgnization level proximity identifier
`beacons[major]`              | Group level proximity identifier
`beacons[minor]`              | Physical beacion proximity identifier
`beacons[links][attributes]`  | List of attribute IDs that belong to the beacon

Curl Example:

```
% curl -s \
       -X GET \
       -H 'Authorization: Token token="secret"' \
       -H "Content-Type: application/vnd.api+json" \
       http://proximitykit.com/api/v1/beacons/6
```

## Creating a beacon

To create a beacon make a POST request to the `/api/v1/beacons/:id`

Parameters

Parameter | Description                  | &nbsp;
--------- | -----------                  | ------
`kit_id`  | The ID of the associated kit | Required
`uuid`    | The beacon UUID              | Required
`name`    | The name of the beacon       |
`major`   | The major beacon identifier  |
`minor`   | The minor beacon identifier  |

`POST /api/v1/beacons`

The response will include the JSON representation of the newly created kit as well as a `Location` header with the URI of the resource.

Example:

```json
{
  "links": {
    "beacons.attributes": "http://proximitykit.com/api/v1/beacon_attributes/{beacons.attributes}"
  },
  "beacons": [
    {
      "id": 6,
      "name": "Earth",
      "uuid": "E58171D9-8398-4453-A991-5593682DDB56",
      "major": 1000,
      "minor": 1,
      "links": {
        "attributes": []
      }
    }
  ]
}
```

Curl Example:

```
% curl -sv \
     -X POST \
     -H 'Authorization: Token token="secret"' \
     -H "Content-Type: application/vnd.api+json" \
     -d '{"beacons": [{"kit_id":52, "name": "Earth 2", "uuid": "E58171D9-8398-4453-A991-5593682DDB56", "major": 1002, "minor": 1}]}' \
     http://proximitykit.com/api/v1/beacons
< HTTP/1.1 201 Created
< Location: http://proximitykit.com/api/v1/beacons/7
```

## Updating an beacon

To update a beacon make a put request `/api/v1/beacons/:id`

Parameters

Parameter | Description                  | &nbsp;
--------- | ------------                 | ------
`kit_id`  | The ID of the associated kit | Required
`uuid`    | The beacon UUID              |
`name`    | The name of the beacon       |
`major`   | The major beacon identifier  |
`minor`   | The minor beacon identifier  |


`PUT /api/v1/beacon/{beacon_id}`

Example:

```
{
  "beacons": [
    {
      "kit_id": 1,
      "name": "My New Name",
      "uuid": "d16eae19-6fce-4198-a5a5-469c9599b709",
      "major": 1,
      "minor": 1
    }
  ]
}
```


## Deleting a Beacon

To delete a beacon make a delete request to `api/v1/beacon/:id`.

`DELETE /api/v1/beacon/:id`

Curl Example:

```
% curl -sv \
       -X DELETE \
       -H 'Authorization: Token token="secret"' \
       -H "Content-Type: application/vnd.api+json" \
       http://proximitykit.com/api/v1/beacons/7
< HTTP/1.1 204 No Content
```

# Beacon Attributes

## Beacon Attributes Resource

`GET /api/v1/beacon_attributes/:id`

This will return a list of beacon attributes.

```
{
  "beacon_attributes": [
    {
      "id": 4,
      "key": "population",
      "value": "eleventy billion"
    }
  ]
}
```

Curl Example:

```
% curl -s \
       -X GET \
       -H 'Authorization: Token token="secret"' \
       -H "Content-Type: application/vnd.api+json" \
       http://proximitykit.com/api/v1/beacon_attributes/4
```

## Creating an attribute

To create an attribute make a post request to `api/v1/beacon_attributes`

Parameters

Parameter   | Description                      | &nbsp;
---------   | ------------                     | ------
`beacon_id` | The ID of the associated beacon  | Required
`key`       | The key value                    |
`value`     | The data associated with the key |


`POST /api/v1/beacon_attributes`

Example:

```json
{
  "beacon_attributes": [
    {
      "beacon_id": 5,
      "key": "venue",
      "value": "spaceship"
    }
  ]
}
```

```
% curl -sv \
       -X POST \
       -H 'Authorization: Token token="secret"' \
       -H "Content-Type: application/vnd.api+json" \
       -d '{ "beacon_attributes": [ { "beacon_id": 5, "key": "venue", "value": "spaceship" } ]}' \
       http://proximitykit.com/api/v1/beacon_attributes
< HTTP/1.1 201 Created
< Location: http://proximitykit.com/api/v1/beacon_attributes/5.5
```

## Updating an attribute

To update an attribute make a post request to `api/v1/beacon_attributes/:id`.

Parameters

Parameter    | Description                      | &nbsp;
---------    | ------------                     | ------
`beacon_id`  | The ID of the parent beacon      | Required
`key`        | The key value                    | Required
`value`      | The data associated with the key | Required

`PUT /api/v1/beacon_attributes/:id`

Example:

```json
{
  "beacon_attributes": [
    {
      "key": "venue",
      "value": "spaceship with dinosaurs"
    }
  ]
}
```

Curl Example:

```
% curl -sv \
       -X PUT \
       -H 'Authorization: Token token="secret"' \
       -H "Content-Type: application/vnd.api+json" \
       -d '{ "beacon_attributes": [ { "key": "venue", "value": "spaceship with dinosaurs" } ]}' \
       http://proximitykit.com/api/v1/beacon_attributes/5
< HTTP/1.1 204 No Content
```

## Deleting an attribute

To delete an attribute make a delete request to the individual `beacon_attribute_url`.

`DELETE /api/v1/beacon_attributes/:id`

Curl Example:

```
% curl -sv \
       -X DELETE \
       -H 'Authorization: Token token="secret"' \
       -H "Content-Type: application/vnd.api+json" \
       http://proximitykit.com/api/v1/beacon_attributes/4
< HTTP/1.1 204 No Content
```
