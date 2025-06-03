# Google Drive Upload

A simple GitHub action to upload files to Google Drive (including support for large files).

### What does this action do better than others?

- Works with large files.
- Can upload multiple files using a wildcard (`my/path/*.txt`).
- Lightweight, with a cold start time of less than 15 seconds.
- Works on Windows, Linux, and Mac.
- Backed by a well-known and widely used upload library (RClone).

#### If you find this project useful, please give it a star ⭐!

## Usage example

### Simple
```yml
- name: Upload to Google Drive
  uses: samuelint/google-drive-upload@v1.1.3
  with:
    folder_id: <your folder id>
    service_account: ${{ secrets.GOOGLE_DRIVE_SERVICE_ACCOUNT }}
    source_path: your_file.txt
```

### Multiple Files (using globstar)
```yml
- name: Upload to Google Drive
  uses: samuelint/google-drive-upload@1.1.3
  with:
    folder_id: <your folder id>
    service_account: ${{ secrets.GOOGLE_DRIVE_SERVICE_ACCOUNT }}
    source_path: |
      some/file/**/some/**/*.dmg
      some/file/**/some/**/*.msi
```

### Large File
```yml
- name: Upload to Google Drive
  uses: samuelint/google-drive-upload@v1.1.3
  with:
    folder_id: <your folder id>
    service_account: ${{ secrets.GOOGLE_DRIVE_SERVICE_ACCOUNT }}
    source_path: your_large_file.zip
    upload_cutoff: 10G # <Optional> Set to avoid HTTP 429 on large file upload
```

## Inputs

### `folder_id`

Required: **YES**.

The **ID of the folder** you want to upload to.

##### How to Find the `Folder ID`

To find the `Folder ID`, look at the URL of your Google Drive folder. For example:

```
https://drive.google.com/drive/folders/xxxx
```

Here, the `Folder ID` is the part after `/folders/`, which in this case is `xxxx`.

### `service_account`

Required: **YES**.

A base64-encoded string of the Google Cloud Service Account Key.

1. Access the `Google Cloud` `Credentials` page:
   https://console.cloud.google.com/apis/credentials
2. Create a new `Service Account`.
3. Generate a new `JSON` key.
4. Convert the key to base64 using:
   ```bash
   cat service_account_key.json | base64 > service_account_key_base64.txt
   ```
5. Add the base64 string as a secret in your GitHub account (or organization).

### `source_path`

Required: **YES**.

The local path to the file(s) to upload.

### `destination_path`

Required: **NO**.

The destination path in Google Drive.

### `upload_cutoff`

Required: **NO**.

The cutoff size for switching to chunked uploads. This allows large files to be uploaded without encountering HTTP 429 errors.


### Windows || Linux || Mac
Use 2 steps with artofact upload strategy.
1. Produce the binary (in the OS you want)
2. Upload

```yml
name: Release

on:
  release:
    types: [published]

jobs:
  build_my_app:
    name: Build my app For Release
    environment: release
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: <some build command>

      - name: Upload Artifacts
        uses: actions/upload-artifact@v4
        with:
          name: ui-dist
          path: dist/*.zip
          retention-days: 1

  upload_to_gdrive_windows:
    name: Upload to Google Drive
    needs: build_my_app
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Download Artifacts
        uses: actions/download-artifact@v4
        with:
          name: ui-dist
          path: dist

      - name: Upload to Google Drive
        uses: samuelint/google-drive-upload@v1.1.3
        with:
          folder_id: <your folder id>
          service_account: ${{ secrets.GOOGLE_DRIVE_SERVICE_ACCOUNT }}
          source_path: dist/*.zip
          upload_cutoff: 10G # <Optional> Set to avoid HTTP 429 on large file upload
```
