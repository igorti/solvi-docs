---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:


includes:


search: false
---

# Introduction

Solvi API is organized around REST and allows for programmatic access to resources like users and projects. Bellow you will find detailed information on how to access the API and which operations are currently supported.

In Solvi, a Project represents a single upload of multiple images. Projects can be grouped under Fields, which in turn are grouped into Farms to make it easier to organize and share data. Current API implementation allows to create and fetch projects for specific user. It is assumed that user is redirected to Solvi in order to upload, process and analyze the imagery.

# Authentication

> > Example request:

```shell
curl "api_endpoint_here"
  -H "Authorization: Bearer <your-jwt-token>"
```

> Make sure to replace `<your-jwt-token>` with your API key.

To start using Solvi API you'll need an access token. We use [JSON Web Tokens](https://jwt.io)(JWT) as token format and expect it to be sent along with every request in Authorization header that looks like following:

`Authorization: Bearer <your-jwt-token>`

Notice that token you receive from us is essentially only needed in order to register new users in Solvi. All user-specific requests, like creating or getting projects, should use a token generated for that specific user. That token is later also used for passwordless authentication when user is redirected from your portal to Solvi.

API is currently open to selected partners only, please [contact us](mailto:support@solvi.nu) in order to receive a token.

# Users

## Register new user

> Example request:

```shell
curl -X POST
  -H "Authorization: Bearer <your-jwt-token>"
  -H "Content-Type: application/json"
  -d '{ "user": { "email": "john@example.com", "password": "password", "first_name": "John", "last_name": "Doe"}}'
  "https://solvi.nu/api/v1/users"
```

> Example response:

```json
  {
    "status": "success",
    "user_id": 182
  }
```

This endpoint registers new user. If successful, the response will return `user_id` that you would later to generate user-specific tokens

### HTTP Request

`POST https://solvi.nu/api/v1/users`

### Parameters

Parameter |  | Description
--------- | ------- | -----------
email | required | Email
password | required | Password
first_name | required | First name
last_name | required | Last name

<aside>
Parameters above must be wrapped into `user` attribute and sent as JSON payload in POST request, see example.
</aside>

## Generate user-specific token

> Example request:

```shell
curl -X GET
  -H "Authorization: Bearer <your-jwt-token>"
  "https://solvi.nu/api/v1/users/<user_id>/token"
```

> Example response:

```json
  {
    "status": "success",
    "user_id": 26,
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyNiwiZXhwIjo0OCwiZXhwIjoxNTAxMjU2NzAyfQ.cu5zIye7ubBhv7YsFIxXkO0E_W0hG0VrlOTQx6L3b3c"
  }
```

This endpoint gives a token for specific user that should be used to create and retrieve user projects. Every request will generate new token. Token is valid for 24 hours.

### HTTP Request

`GET https://solvi.nu/api/v1/users/<user_id>/token`

### Parameters

Parameter |  | Description
--------- | ------- | -----------
user_id | required | User ID given when user is created

# Fields

## Create field

> Example request:

```shell
curl -X POST
  -H "Authorization: Bearer <user-jwt-token>"
  -H "Content-Type: application/json"
  -d '{"field": { "name": "Winter Wheat", "geom": "{\"type\":\"Polygon\",\"coordinates\":[[[100.0, 0.0],[101.0, 0.0],[101.0, 1.0],[100.0, 1.0],[100.0, 0.0]]]}"}}'
  "https://solvi.nu/api/v1/fields"
```

> Example response:

```json
  {
    "status": "success",
    "field_id": 793
  }
```

Creates new field. Field boundaries can be provided as Polygon or MultiPolygon in GeoJSON format. The response contains `field_id` which can be later used to relate projects to specific field.

### HTTP Request

`POST https://solvi.nu/api/v1/fields`

### Parameters

Parameter | | Description
--------- | ----------- | -----------
name | | Name of the the field
geom | optional | Boundaries of the field as a Polygon or Multipolygon in [GeoJSON format](https://geojson.org/geojson-spec.html#introduction) and EPSG:4326 coordinate system(lonlat)


## Get fields

> Example request:

```shell
curl -X GET
  -H "Authorization: Bearer <user-jwt-token>"
  -H "Content-Type: application/json"
  "https://solvi.nu/api/v1/fields"
```

> Example response:

```json
    [
      {
          "name": "wheat field",
          "created_at": "2018-02-27T12:45:05.550Z",
          "projects": [
              {
                  "project_id": 1291,
                  "status": "processed",
                  "name": "27-FEB-2018",
                  "field_name": "Wheat field",
                  "survey_date": "2018-02-26T14:19:36.000Z",
                  "upload_date": "2018-02-27T12:45:05.556Z",
                  "url": "https://solvi.nu/projects/1291",
                  "thumbnail_url": "https://solvi.nu/projects/1291/thumbnail.png"
              }
          ]
      }
    ]
```

Gets all user fields and a list of projects related to each field.

### HTTP Request

`GET https://solvi.nu/api/v1/fields`


# Projects

## Create project

> Example request:

```shell
curl -X POST
  -H "Authorization: Bearer <user-jwt-token>"
  -H "Content-Type: application/json"
  -d '{ "field_id": "field_1", "field_name": "Wheat Field", "field_geom": "{\"type\":\"Polygon\",\"coordinates\":[[[100.0, 0.0],[101.0, 0.0],[101.0, 1.0],[100.0, 1.0],[100.0, 0.0]]]}" }'
  "https://solvi.nu/api/v1/projects"
```

> Example response:

```json
  {
    "status": "success",
    "project_id": 142,
    "project_url": "https://solvi.nu/projects/142/photos/upload"
  }
```

This endpoint creates a new project which is required prior to imagery upload. In response you will receive url to upload page for newly created project where user can be redirected.

Projects can be connected to a Field. When multiple projects are related to the same Field, they appear in the same map view when data is processed. This allows for easier navigation between imagery over the same Field and over the time data comparison.

Fields can be created either beforehand - then `field_id` parameter should be specified when creating a project, or on the fly by sending in `field_name` and `field_geom`.

### HTTP Request

`POST https://solvi.nu/api/v1/projects`

### Parameters

Parameter | | Description
--------- | ----------- | -----------
field_id | | Unique field identifier
field_name | optional | Name of the the field
field_geom | optional | Boundaries of the field as a polygon in [GeoJSON format](https://geojson.org/geojson-spec.html#introduction) and EPSG:4326 coordinate system(lonlat)


## Get projects

> Example request:

```shell
curl -X GET
  -H "Authorization: Bearer <user-jwt-token>"
  'https://solvi.nu/api/v1/projects?field_geom={"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Polygon","coordinates":[[[100.0, 0.0],[101.0, 0.0],[101.0, 1.0],[100.0, 1.0],[100.0, 0.0]]]}]}'
```

> Example response:

```json
  [
    {
        "project_id": 9999,
        "status": "processed",
        "name": "25-JUN-2018",
        "field": {
            "id": 9999,
            "name": "Winter Wheat",
            "identfier": "WW-01",
            "farm": {
                "id": 656,
                "name": "Borgeby Farm"
            }
        },
        "survey_date": "2018-06-25T19:19:27.000Z",
        "upload_date": "2018-06-25T20:50:14.870Z",
        "url": "https://solvi.nu/projects/9999",
        "thumbnail_url": "http://solvi.nu/projects/9999/thumbnail.png"
    },
    {
      "name": "New project",
      "url": "https://solvi.nu/projects/new"
    }
  ]
```

This endpoint retrieves all projects created by the user or shared with user by others. If field boundaries are provided, only projects whose extent overlaps boundaries are returned.

### HTTP Request

`GET https://solvi.nu/api/v1/projects`

### Parameters

Parameter | | Description
--------- | ----------- | -----------
field_geom | optional | Boundaries of the field as a polygon in [GeoJSON format](https://geojson.org/geojson-spec.html#introduction) and EPSG:4326 coordinate system(lonlat).

## Project outputs

There are several webhooks in Solvi that allow user generated outputs, like prescription files, to be exported or even posted back to partner API if it's available. This is handled on per case basis at the moment, [contact us](mailto:support@solvi.nu) for more details.
