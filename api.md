## Get User Profile

Request the user's profile information. This will return the user's profile information, including their user ID, email, and other information. Requires the `profile:read` scope.

```http
GET https://api.fascatapi.com/public/v1/profile
Authorization: Bearer <access_token>
```

Response:

```json
{
  "uid": "a1b2c3d4e5f6",
  "email": "test@example.com",
  "name": "Test User"
}
```

- `uid` - The user's FasCat unique ID.
- `email` - The user's email address.
- `name` - The user's name. May be `null` if the user has not set a name.

## Get Workouts

Request the user's workouts. This will return workouts with the `steps` in the requested format. Requires the `workout:read` scope.

```http
GET https://api.fascatapi.com/public/v1/workouts?start=<startDate>&end=<endDate>&format=zwo
Authorization: Bearer <access_token>
```

Parameters:

- `start` - The start date of the range to get workouts for. Must be in ISO 8601 date format. Ex: `2024-09-01`.
- `end` - The end date of the range to get workouts for. Must be in ISO 8601 date format. Ex: `2024-09-03`.
- `format` - The format to return the workouts in. Currently only `zwo` is supported. Can leave off the format query param and nothing will be returned for the workout's `steps`.

Response:

```json
{
  "uid": "a1b2c3d4e5f6",
  "workouts": [
    {
      "id": "2fb696cdbd2847acabbf4c2ff8fc8f22",
      "date": "2024-07-09",
      "type": "cycle",
      "title": "Field Test: 20 minuter then repeat (2 x 20 workout)",
      "description": "20 Minute threshold effort...",
      "duration": 5700,
      "thresholds": {
        "power": 236,
        "hr": 165
      },
      "format": "zwo",
      "steps": "<workout_file><author>FasCat</author><name>Field Test: 20 minutes..."
    },
    ...
  ]
}
```

Properties:

- `uid` - The user's FasCat unique ID.
- `workouts` - An array of the user's workouts for the requested date range.
  - `id` - The workout unique ID.
  - `date` - The workout date in ISO 8601 format.
  - `type` - The workout type. The API currently filters out everything except `cycle`, and `mtb` workouts.
  - `title` - The workout's title.
  - `description` - The workout's description.
  - `duration` - The workout's duration in seconds.
  - `thresholds` - The user's current sport type specific thresholds. For cycling and mountain biking, this includes:
    - `power` - The workout's power threshold.
    - `hr` - The workout's heart rate threshold.
  - `format` - The workout's format. Ex: `zwo`.
  - `steps` - The workout's steps in the requested format. For `zwo` format, this is the ZWO XML file contents as a string. Will be `null` if the `format` query param is not provided.

## Upload Completed Activities

The FasCat API allows applications to upload completed activities. This is useful for automatically syncing completed activities from other platforms to FasCat. Requires the `activity:write` scope. This is a 2-step process. First, the app must initiate the activity upload. Then, the app must upload the activity file to the returned URL using `HTTP PUT`.

### Initiate Activity Upload

```http
POST https://api.fascatapi.com/public/v1/activity/initiate-upload
Authorization: Bearer <access_token>
Content-Type: application/json

{
    "title": "Test Activity",
    "type": "fit",
    "externalActivityId": "123456"
}
```

Input:

- `title` - The activity title.
- `type` - The activity file type. Currently only `fit` is supported.
- `externalActivityId` - The external activity ID. Used to prevent duplicate uploads.

Response:

```json
{
  "id": "5e6fa65ba8b04667b9cf879ee290e8cc",
  "fileUploadUrl": "https://prod-fascat-device.s3.amazonaws.com/upload/pending/a1b2c3d4e5f6/activity/..."
}
```

- `id` - The ID of the upload request. Unused at the moment. Future APIs may use this for checking the status of the upload and file processing progress.
- `fileUploadUrl` - The URL to upload the activity file to. This URL is temporary and will expire after a short period of time.

### Upload Activity File

Upload the activity file to the returned URL using `HTTP PUT`. The file must be uploaded as`binary/octet-stream`. No authorization is required for this request.

```
PUT <fileUploadUrl>
Content-Type: binary/octet-stream

<binary_data>
```
