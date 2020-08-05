---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:


includes:


search: false
---

# Introduction

Solvi API is organized around REST and allows for programmatic access to resources like users and projects. Below you will find detailed information on how to access the API and which operations are currently supported.

In Solvi, a Project represents a single upload of multiple images. Projects can be grouped under Fields, which in turn are grouped into Farms to make it easier to organize and share data. Current API implementation allows to create and fetch projects for specific user. It is assumed that user is redirected to Solvi in order to upload, process and analyze the imagery.

# Authentication

> Example API key request:

```shell
curl "api_endpoint_here"
  -H "X-Api-Key: <your-api-key>"
```
> Make sure to replace `<your-api-key>` with your API key.

To use Solvi API you first need an *API key*. You will get an API key from Solvi, [contact us](mailto:support@solvi.nu) to get your key. If you for some
reason need to revoke a key, you can also contact us.

Please note that the API key should be kept secret and not be exposed to end users: for example, never send the API key to a web browser.

With the API key, you can access general API methods, for example to [register new users](#register-new-user) in Solvi, and create user specific tokens.

Requests requiring an API key should add the `X-Api-Key` header:

`X-Api-Key: <your-api-key>`

> Example user specific request:

```shell
curl "api_endpoint_here"
  -H "Authorization: Bearer <user-specific-token>"
```
> Make sure to replace `<user-specific-token>` with your token.

All user-specific requests, like creating or getting projects, should use a user specific token, which has a limited lifetime and can therefor be used directly from a browser, or for passwordless authentication when user is redirected from your portal to Solvi. A user specific token can be used for a limited amount of time (currently 24 hours) before it expires. A user specific token is created by using the [endpoint to create user-specific token](#generate-user-specific-token).

User specific requests should use the `Authorization` header *instead of* the `X-Api-Key` header:

`Authorization: Bearer <user-specific-token>`

# Users

## Register new user

> Example request:

```shell
curl -X POST
  -H "X-Api-Key: <your-api-key>"
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

This endpoint registers new user. If successful, the response will return `user_id` that you would use later to generate user-specific tokens

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
  -H "X-Api-Key: <your-api-key>"
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
  -d '{ "type": "overlapping", "field_id": "field_1", "field_name": "Wheat Field", "field_geom": "{\"type\":\"Polygon\",\"coordinates\":[[[100.0, 0.0],[101.0, 0.0],[101.0, 1.0],[100.0, 1.0],[100.0, 0.0]]]}" }'
  "https://solvi.nu/api/v1/projects"
```

> Example response:

```json
  {
    "status": "success",
    "project_id": 142,
    "project_url": "https://solvi.nu/projects/142/photos/upload",
    "imagery_upload_data": {
      "url":"https://solvi-projects-dev.s3.eu-west-1.amazonaws.com",
      "fields": {
        [...]
      },
      "key_prefix":"uploads/26c1ce11-c9bf-4825-821c-72e9f600a6cf/originals/"
    }
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
type      | optional  | The type of imagery for this project: `overlapping` or `stitched`; default is `overlapping`
field_id | | Unique field identifier
field_name | optional | Name of the the field
field_geom | optional | Boundaries of the field as a polygon in [GeoJSON format](https://geojson.org/geojson-spec.html#introduction) and EPSG:4326 coordinate system(lonlat)

## Upload project imagery

> Example request:

```shell
curl -X POST
  -H "Authorization: Bearer <user-jwt-token>"
  -H "Content-Type: application/json"
  "https://solvi.nu/api/v1/projects/<project_id>/begin_upload"
```

> Example response:

```json
{
  "status": "ok",
  "upload_imagery_data": {
    "url": "https://solvi-projects.s3.eu-west-1.amazonaws.com",
    "fields": {
      "x-amz-storage-class": "STANDARD_IA",
      "policy": "eyJleHBpcmF0aW9uIjoiMjAyMC0wOC0wN1QwNzozMjo1MVoiLCJjb25kaXRpb25zIjpbeyJidWNrZXQiOiJzb2x2aS1wcm9qZWN0cy1kZXYifSxbInN0YXJ0cy13aXRoIiwiJGtleSIsInVwbG9hZHMvMjZjMWNlMTEtYzliZi00ODI1LTgyMWMtNzJlOWY2MDBhNmNmL29yaWdpbmFscy8iXSxbInN0YXJ0cy13aXRoIiwiJENvbnRlbnQtVHlwZSIsIiJdLHsieC1hbXotc3RvcmFnZS1jbGFzcyI6IlNUQU5EQVJEX0lBIn0seyJ4LWFtei1jcmVkZW50aWFsIjoiQUtJQVVWTUhLR1RRRldMTlkyTUgvMjAyMDA4MDUvZXUtd2VzdC0xL3MzL2F3czRfcmVxdWVzdCJ9LHsieC1hbXotYWxnb3JpdGhtIjoiQVdTNC1ITUFDLVNIQTI1NiJ9LHsieC1hbXotZGF0ZSI6IjIwMjAwODA1VDA3MzI1MVoifV18",
      "x-amz-credential": "AKIAUVMHKGTQFWLNY2MX/20200805/eu-west-1/s3/aws4_request",
      "x-amz-algorithm": "AWS4-HMAC-SHA256",
      "x-amz-date": "20200805T073251Z",
      "x-amz-signature": "5494b8f7ca5c78c45daf7d8099f3c4b93e141567d3691dbec3dcc0f6fd2e0181"
    },
    "key_prefix": "uploads/26c1ce11-c9bf-4825-821c-72e9f600a6cf/originals/"
  }
}
```

After a project has been created, imagery can be uploaded to it by sending the `begin_upload` request. The response includes information required to upload
imagery for the project.

> Example JavaScript function to POST an image using `upload_imagery_data`:

```js
  function uploadFile (upload_imagery_data, f) {
    const form = new FormData()
    Object.keys(upload_imagery_data.fields).forEach(field => form.append(field, upload_imagery_data.fields[field]))
    form.append('key', upload_imagery_data.key_prefix + f.name)
    form.append('Content-Type', f.type)
    form.append('file', f)

    return fetch(postInfo.url, { method: 'POST', body: form })
  }
```

The required parameters are included in the `upload_imagery_data` object: this object has a `url` property indicating the URL to POST imagery to, and a `fields` object, listing the HTTP form data fields required for the POST request. In addition to these fields, the form must also include a `key` field: the key is the must include a value prefixed by the `key_prefix` and a value unique for each image (like its filename).

### HTTP Request

`POST https://solvi.nu/api/v1/projects/<project_id>/begin_upload`

### Parameters

None.

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
