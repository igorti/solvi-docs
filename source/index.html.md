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

In Solvi, a Project represents a single upload of multiple images. Projects can be grouped under Fields, which in turn are grouped into Farms to make it easier to organize and share data. Current API implementation allows to create and fetch projects for the specific user, as well as upload and stitch images into maps. Once processed, the outputs such as orthomosaics and elevation maps can either be accessed through the API or the user can be redirected to Solvi to view and analyze the processed imagery.

# Authentication

> Example API key request:

```shell
curl "api_endpoint_here"
  -H "X-Api-Key: <your-api-key>"
```
> Make sure to replace `<your-api-key>` with your API key.

To use Solvi API you first need an *API key*. You will get an API key from Solvi, [contact us](mailto:support@solvi.nu) to get your key. If you for some
reason need to revoke a key, you can also contact us.

Please note that the API key should be kept secret and not be exposed to end-users: for example, never send the API key to a web browser.

With the API key, you can access general API methods, for example, to [register new users](#register-new-user) in Solvi, and create user-specific tokens.

Requests requiring an API key should add the `X-Api-Key` header:

`X-Api-Key: <your-api-key>`

> Example user-specific request:

```shell
curl "api_endpoint_here"
  -H "Authorization: Bearer <user-specific-token>"
```
> Make sure to replace `<user-specific-token>` with your token.

All user-specific requests, like creating or getting projects, should use a user-specific token, which has a limited lifetime and can therefore, be used directly from a browser, or for passwordless authentication when the user is redirected from your portal to Solvi. A user-specific token can be used for a limited amount of time (currently 24 hours) before it expires. A user-specific token is created by using the [endpoint to create a user-specific token](#generate-user-specific-token).

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
  "https://solvi.ag/api/v1/users"
```

> Example response:

```json
  {
    "status": "success",
    "user_id": 182
  }
```

This endpoint registers a new user. If successful, the response will return `user_id` that you would use later to generate user-specific tokens.

### HTTP Request

`POST https://solvi.ag/api/v1/users`

### Parameters

Parameter |  | Description
--------- | ------- | -----------
email | required | Email
password | required | Password
first_name | required | First name
last_name | required | Last name

<aside>
Parameters above must be wrapped into `user` attribute and sent as JSON payload in POST request, see the example.
</aside>

## Generate user-specific token

> Example request:

```shell
curl -X GET
  -H "X-Api-Key: <your-api-key>"
  "https://solvi.ag/api/v1/users/<user_id>/token"
```

> Example response:

```json
  {
    "status": "success",
    "user_id": 26,
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyNiwiZXhwIjo0OCwiZXhwIjoxNTAxMjU2NzAyfQ.cu5zIye7ubBhv7YsFIxXkO0E_W0hG0VrlOTQx6L3b3c"
  }
```

This endpoint gives a token for the specific user that should be used to create and retrieve user projects. Every request will generate na ew token. The token is valid for 24 hours.

### HTTP Request

`GET https://solvi.ag/api/v1/users/<user_id>/token`

### Parameters

Parameter |  | Description
--------- | ------- | -----------
user_id | required | User ID given when user is created

# Farms

## Create farm

> Example request:

```shell
curl -X POST
  -H "Authorization: Bearer <user-jwt-token>"
  -H "Content-Type: application/json"
  -d '{"farm": {"name": "My second farm"}}'
  "https://solvi.ag/api/v1/farms"
```

> Example response:

```json
  {
    "status": "success",
    "farm_id": 793
  }
```

Creates a new farm.

### HTTP Request

`POST https://solvi.ag/api/v1/farms`

### Parameters

Parameter | | Description
--------- | ----------- | -----------
name | | Name of the the farm

## Get farms

> Example request:

```shell
curl -X GET
  -H "Authorization: Bearer <user-jwt-token>"
  -H "Content-Type: application/json"
  "https://solvi.ag/api/v1/farms"
```

> Example response:

```json
    [
      {
        "name": "My farm",
        "created_at": "2019-02-08T09:18:37.655Z",
        "fields": [
          {
            "id": 2657,
            "name": "Veddige",
            "created_at": "2019-03-08T04:05:38.628Z"
          }
        ]
      }
    ]
```

Gets all user farms and a list of fields related to each farm.

### HTTP Request

`GET https://solvi.ag/api/v1/fields`


# Fields

## Create field

> Example request:

```shell
curl -X POST
  -H "Authorization: Bearer <user-jwt-token>"
  -H "Content-Type: application/json"
  -d '{"field": { "name": "Winter Wheat", "geom": "{\"type\":\"Polygon\",\"coordinates\":[[[100.0, 0.0],[101.0, 0.0],[101.0, 1.0],[100.0, 1.0],[100.0, 0.0]]]}"}}'
  "https://solvi.ag/api/v1/fields"
```

> Example response:

```json
  {
    "status": "success",
    "field_id": 793
  }
```

Creates a new field. Field boundaries can be provided as Polygon or MultiPolygon in GeoJSON format. The farm to organize the field under can also be provided. The response contains `field_id` which can be later used to relate projects to the specific field.

### HTTP Request

`POST https://solvi.ag/api/v1/fields`

### Parameters

Parameter | | Description
--------- | ----------- | -----------
name | | Name of the the field
geom | optional | Boundaries of the field as a Polygon or Multipolygon in [GeoJSON format](https://geojson.org/geojson-spec.html#introduction) and EPSG:4326 coordinate system(lonlat)
farm_id | optional | The id of the farm to put the field under; if not specified, the user's last created farm is used


## Get fields

> Example request:

```shell
curl -X GET
  -H "Authorization: Bearer <user-jwt-token>"
  -H "Content-Type: application/json"
  "https://solvi.ag/api/v1/fields"
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
                  "url": "https://solvi.ag/projects/1291",
                  "thumbnail_url": "https://solvi.ag/projects/1291/thumbnail.png"
              }
          ]
      }
    ]
```

Gets all user fields and a list of projects related to each field.

### HTTP Request

`GET https://solvi.ag/api/v1/fields`


# Projects

## Create project

> Example request:

```shell
curl -X POST
  -H "Authorization: Bearer <user-jwt-token>"
  -H "Content-Type: application/json"
  -d '{ "type": "overlapping", "field_id": "field_1", "field_name": "Wheat Field", "field_geom": "{\"type\":\"Polygon\",\"coordinates\":[[[100.0, 0.0],[101.0, 0.0],[101.0, 1.0],[100.0, 1.0],[100.0, 0.0]]]}" }'
  "https://solvi.ag/api/v1/projects"
```

> Example response:

```json
  {
    "status": "success",
    "project_id": 142,
    "project_url": "https://solvi.ag/projects/142/photos/upload",
    "imagery_upload_data": {
      "url":"https://solvi-projects-dev.s3.eu-west-1.amazonaws.com",
      "fields": {
        [...]
      },
      "key_prefix":"uploads/26c1ce11-c9bf-4825-821c-72e9f600a6cf/originals/"
    }
  }
```

This endpoint creates a new project which is required before imagery upload. In response, you will receive URL to upload-page for the newly created project where the user can be redirected.

Projects can be connected to a Field. When multiple projects are related to the same Field, they appear in the same map view when data is processed. This allows for easier navigation between imagery over the same Field and over the time data comparison.

Fields can be created either beforehand - then `field_id` parameter should be specified when creating a project, or on the fly by sending in `field_name` and `field_geom`.

Optionally, a project can be created with a so-called *webhook* that will be called every time the status of the project changes. This makes it possible for integration to for example react when a project finishes processing, without having to use polling to check the project's status. To add a webhook, specify the URL to be called with the `status_webhook` parameter. See the section on [webhooks](#webhooks) for details.

### HTTP Request

`POST https://solvi.ag/api/v1/projects`

### Parameters

Parameter | | Description
--------- | ----------- | -----------
type      | optional  | The type of imagery for this project: `overlapping` or `stitched`; default is `overlapping`
field_id | | Unique field identifier
field_name | optional | Name of the the field
field_geom | optional | Boundaries of the field as a polygon in [GeoJSON format](https://geojson.org/geojson-spec.html#introduction) and EPSG:4326 coordinate system(lonlat)
status_webhook | optional | A URL to be called when the project's status changes
webhook_secret | optional | A token to use to sign webhook requests, see [webhooks](#webhooks) for details

## Upload project imagery

> Example request:

```shell
curl -X POST
  -H "Authorization: Bearer <user-jwt-token>"
  -H "Content-Type: application/json"
  "https://solvi.ag/api/v1/projects/<project_id>/begin_upload"
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

> Example JavaScript function to POST an image using `upload_imagery_data`
> retrieved from the example above:

```js
function uploadFile (upload_imagery_data, f) {
  const form = new FormData()
  Object.keys(upload_imagery_data.fields).forEach(field => form.append(field, upload_imagery_data.fields[field]))
  const key = upload_imagery_data.key_prefix + f.name
  form.append('key', key)
  form.append('Content-Type', 'image/jpeg')
  form.append('file', f)

  return fetch(upload_imagery_data.url, { method: 'POST', body: form })
    .then(response => {
      if (!response.ok) {
        throw new Error(`Unexpected response HTTP ${response.status} ${response.statusText}`)
      }
    })
}
```

> Example JavaScript function to post an array of files using the `uploadFile`
> function above, using `upload_imagery_data`
> retrieved from the example above:

```js
function uploadFiles (upload_imagery_data, files) {
  return uploadNext()

  function uploadNext () {
    return new Promise(function (resolve, reject) {
      const file = files.shift()
      if (file) {
        uploadFile(upload_imagery_data, file)
          .then(uploadNext)
          .catch(reject)
      } else {
        resolve()
      }
    })
  }
}
```


The required parameters are included in the `upload_imagery_data` object: this object has a `url` property indicating the URL to POST imagery to, and a `fields` object, listing the HTTP form data fields required for the POST request. In addition to these fields, the form must also include a `key` field: the key is the must include a value prefixed by the `key_prefix` and value unique for each image (like its filename).

### HTTP Request

`POST https://solvi.ag/api/v1/projects/<project_id>/begin_upload`

### Parameters

Parameter |  | Description
--------- | ------- | -----------
project_id | required | Project ID given when project is created

## Processing uploaded imagery

> Example request:

```shell
curl -X POST
  -H "Authorization: Bearer <user-jwt-token>"
  -H "Content-Type: application/json"
  "https://solvi.ag/api/v1/projects/<project_id>/complete_upload"
```

> Example response:

```json
{
  "status": "ok",
}
```

When project imagery has been uploaded, the upload must be completed, which will start the processing of the imagery.

### HTTP Request

`POST https://solvi.ag/api/v1/projects/<project_id>/complete_upload`

### Parameters

Parameter |  | Description
--------- | ------- | -----------
project_id | required | Project ID given when project is created

## Get projects

> Example request:

```shell
curl -X GET
  -H "Authorization: Bearer <user-jwt-token>"
  'https://solvi.ag/api/v1/projects?field_geom={"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Polygon","coordinates":[[[100.0, 0.0],[101.0, 0.0],[101.0, 1.0],[100.0, 1.0],[100.0, 0.0]]]}]}'
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
        "url": "https://solvi.ag/projects/9999",
        "thumbnail_url": "https://solvi.ag/projects/9999/thumbnail.png"
    },
    {
      "name": "New project",
      "url": "https://solvi.ag/projects/new"
    }
  ]
```

This endpoint retrieves all projects created by the user or shared with user by others. If field boundaries are provided, only projects whose extent overlaps boundaries are returned.

### HTTP Request

`GET https://solvi.ag/api/v1/projects`

### Parameters

Parameter | | Description
--------- | ----------- | -----------
field_geom | optional | Boundaries of the field as a polygon in [GeoJSON format](https://geojson.org/geojson-spec.html#introduction) and EPSG:4326 coordinate system(lonlat).

## Project outputs

> Example request:

```shell
curl -X GET
  -H "Authorization: Bearer <user-jwt-token>"
  'https://solvi.ag/api/v1/projects/<project-id>
```

> Example response:

```json
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
      "url": "https://solvi.ag/projects/9999",
      "thumbnail_url": "https://solvi.ag/projects/9999/thumbnail.png",
      "resources": {
        "thumbnail": "https://solvi.ag/projects/9999/thumbnail.png",
        "ortho": "https://solvi-projects.s3.eu-west-1.amazonaws.com/uploads/ed12e5f6-b6c9-4df8-9522-1dbc29be854b/results/ortho.tiff?...",
        "dem": "https://solvi-projects-dev.s3.eu-west-1.amazonaws.com/uploads/ed12e5f6-b6c9-4df8-9522-1dbc29be854b/results/dem.tiff?..."
      },
      "tiles": {
        "ortho": "https://ts1.solvi.nu/ed12e5f6-b6c9-4df8-9522-1dbc29be854b/ortho_rgb/{z}/{x}/{y}.png?token=...",
        "dem": "https://ts1.solvi.nu/ed12e5f6-b6c9-4df8-9522-1dbc29be854b/dem_color/{z}/{x}/{y}.png?token=..."
      }
  },
```

This endpoint retrieves projects created by the user or shared with the user by others.

The included `resources` are URLs that can be used to fetch the project outputs. These resource URLs are temporary, with a lifetime of 15 minutes before they expire.

The response also includes URL templates for tiled maps in the `tiles` section: these URLs can be used directly with several popular map clients like [OpenLayers](https://openlayers.org/) or [Leaflet](https://leafletjs.com/). The URLs are temporary, with a lifetime of at least 48 hours.

### HTTP Request

`GET https://solvi.ag/api/v1/projects/<project_id>`

### Parameters

Parameter |  | Description
--------- | ------- | -----------
project_id | required | Project ID given when project is created

## Webhooks

Webhooks allow setting up integrations that subscribe to certain events from Solvi. When one of those events is triggered, an HTTP POST payload is sent to the webhook's configured URL. Currently, project status is the only available webhook in Solvi.

> Example webhook request payload

```json
{
  "project_id": 8993,
  "event_type": "status_changed",
  "old_status": "created",
  "new_status": "uploading"
}
```

The webhook is configured when [creating a project](#create-project) by specifing the `status_webhook` parameter, which should contain the URL of the webhook.

### Securing webhooks

To ensure that Solvi is the sender of the webhook requests, you can optionally also specify a secret token when registering the webhook, by using the `webhook_secret` parameter: the secret can be any string of your choosing.

> Example signature header

```
X-Solvi-Signature: sha1=494e5dbdd1afbe4d44091bf86872b5eb4b9133e5
```

When a secret token has been specified, Solvi will include the HTTP header `X-Solvi-Signature`, which will contain an HMAC-SHA1 signature of the body.

This follows the same pattern as [securing webhooks on GitHub](https://docs.github.com/en/developers/webhooks-and-events/securing-your-webhooks), except for using the header `X-Solvi-Signature` instead.
