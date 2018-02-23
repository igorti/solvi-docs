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
    "user_id": 26,
    "user_profile_url": "https://solvi.nu/edit"
  }
```

This endpoint registers new user.

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

This endpoint gives a token for specific user that should be used to create and retrieve user projects. Every request will generate new token. Token is valid for 48 hours.

### HTTP Request

`GET https://solvi.nu/api/v1/users/<user_id>/token`

### Parameters

Parameter |  | Description
--------- | ------- | -----------
user_id | required | User ID given when user is created

# Projects

## Create new project

> Example request:

```shell
curl -X POST
  -H "Authorization: Bearer <user-jwt-token>"
  -H "Content-Type: application/json"
  -d '{ "field_id": "field_1", "field_name": "Wheat Field", "field_geom": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Polygon\",\"coordinates\":[[[100.0, 0.0],[101.0, 0.0],[101.0, 1.0],[100.0, 1.0],[100.0, 0.0]]]}]}" }'
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

### HTTP Request

`POST https://solvi.nu/api/v1/projects`

### Parameters

Parameter | | Description
--------- | ----------- | -----------
field_id | optional | Unique field identifier
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
      "project_id": 142,
      "name": "27-JUL-2017",
      "url": "https://solvi.nu/projects/142/photos/upload"
    },
    {
      "name": "New project",
      "url": "https://solvi.nu/projects/new"
    }
  ]
```

This endpoint retrieves all projects for a specific field given the boundaries of the field. If no bounadaries are specified, all projects belonging to the user identified by token are returned.

### HTTP Request

`GET https://solvi.nu/api/v1/projects`

### Parameters

Parameter | | Description
--------- | ----------- | -----------
field_geom | optional | Boundaries of the field as a polygon in [GeoJSON format](https://geojson.org/geojson-spec.html#introduction) and EPSG:4326 coordinate system(lonlat).

## Project outputs

There are several webhooks in Solvi that allow user generated outputs, like prescription files, to be exported or even posted back to partner API if it's available. This is handled on per case basis at the moment, [contact us](mailto:support@solvi.nu) for more details.
