---
layout: post
title: "Uploading a blob to an Azure container using curl"
tags: 
  - bash
  - curl
  - azure
  - container
  - blob
  - s3
  - Microsoft
  - upload
---

I had to upload files to an Azure container (An S3 alternative) on a Raspberry Pi for a recent project.
My first choice was to use [azcopy](https://learn.microsoft.com/en-us/azure/storage/common/storage-ref-azcopy); however, it does not support `armv7`, and after a brief stint at using `binfmt` and `qemu`, I opted to use curl.

As a Microsoft product, there are many bad articles on how to do this, and many are for old API versions.
The snippets below are focused on using the smallest amount of headers for `x-ms-version: 2019-12-12` and files under 100MB.

```bash
  FILE_PATH="/path/to/file"

  HEADER_MS_DATE=$(TZ=GMT date "+%a, %d %h %Y %H:%M:%S %Z")
  HEADER_CONTENT_LENGTH=$(wc --bytes < "${FILE_PATH}") 

  AZURE_CONTAINERS_URL=https://example.blob.core.windows.net"
  AZURE_CONTAINER_NAME=big_box
  AZURE_BLOB_PATH_URI="URI/PATH/FOR/FILE.txt"
  AZURE_CONTAINER_PATH="${AZURE_CONTAINERS_URL}/${AZURE_CONTAINER_NAME}/${AZURE_BLOB_PATH_URI}"

  AZURE_CONTAINER_SAS_TOKEN="sp=racw&st=2022-05-16T17:20:00Z&se=2022-12-05T00:20:00Z&spr=https&sv=2021-01-09&sr=c&sig.."

  curl \
    --request PUT \
    --fail \
    --silent \
    --header "x-ms-version: 2019-12-12" \
    --header "x-ms-date: ${HEADER_MS_DATE}" \
    --header "x-ms-blob-type: BlockBlob" \
    --header "Content-Length: ${HEADER_CONTENT_LENGTH}" \
    --data-binary "@${FILE_PATH}" \
    "${AZURE_CONTAINER_PATH}?${AZURE_CONTAINER_SAS_TOKEN}"
```

If you want to have Azure check your files on upload, you can compute a local `md5sum` and pass it as a header. Azure will create the `md5sum` server side if you do not pass this header.

```bash
  FILE_PATH="/path/to/file"

  HEADER_MS_DATE=$(TZ=GMT date "+%a, %d %h %Y %H:%M:%S %Z")
  HEADER_CONTENT_LENGTH=$(wc --bytes < "${FILE_PATH}") 
  HEADER_CONTENT_MD5=$(md5sum -b "${FILE_PATH}" | awk '{ print $1 }' | xxd -p -r | base64)

  AZURE_CONTAINERS_URL=https://example.blob.core.windows.net"
  AZURE_CONTAINER_NAME=big_box
  AZURE_BLOB_PATH_URI="URI/PATH/FOR/FILE.txt"
  AZURE_CONTAINER_PATH="${AZURE_CONTAINERS_URL}/${AZURE_CONTAINER_NAME}/${AZURE_BLOB_PATH_URI}"

  AZURE_CONTAINER_SAS_TOKEN="sp=racw&st=2022-05-16T17:20:00Z&se=2022-12-05T00:20:00Z&spr=https&sv=2021-01-09&sr=c&sig.."

  curl \
    --request PUT \
    --fail \
    --silent \
    --header "x-ms-version: 2019-12-12" \
    --header "x-ms-date: ${HEADER_MS_DATE}" \
    --header "x-ms-blob-type: BlockBlob" \
    --header "Content-Length: ${HEADER_CONTENT_LENGTH}" \
    --header "Content-MD5: ${HEADER_CONTENT_MD5}" \
    --data-binary "@${FILE_PATH}" \
    "${AZURE_CONTAINER_PATH}?${AZURE_CONTAINER_SAS_TOKEN}"
```

If this was helpful to you, please reach out and let me know.
