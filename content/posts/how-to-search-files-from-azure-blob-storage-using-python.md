---
title: "How to search files from Azure Blob Storage using Python"
draft: false
date: "2022-12-13T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["python", "azure"]
description: "Searching for files using index tags is a common use case but the Azure documentation can be quite hard to grasp. "
---

Searching for files using [index tags](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-index-how-to?tabs=azure-portal) is a common use case but the Azure documentation can be quite hard to grasp.

To search for blobs with specific tags in Azure Blob Storage using Python, you can use the `find_blobs_by_tags` method from the Azure Storage SDK. It takes as a single parameter `filter_expression` which can be used to query blobs with specific tags, container or name.

Here's an example of how to use `find_blobs_by_tags` to search for blobs with specific tags

```python
from azure.storage.blob import BlobServiceClient

# Create a BlobServiceClient object
blob_service_client = BlobServiceClient.from_connection_string("<your_connection_string>")

# Get a reference to the container where your blobs are stored
container_client = blob_service_client.get_container_client("<your_container_name>")

# Use the find_blobs_by_tags method to search for blobs with a specific set of tags
results = container_client.find_blobs_by_tags(
    "key1 = 'value1' and key2 = 'value2'"
)

# Print the names of the blobs that were found
for blob in results:
    print(blob["name"])

```

In this example, the `find_blobs_by_tags` method is used to search for blobs that have the tags `key1` with a value of `value1` and `key2` with a value of `value2`.

`find_blobs_by_tags` returns a iterable response of `FilteredBlob` objects. The return value is automatically paginated and you can control the number of `Blobs` per page with the `results_per_page` keyword argument. You can iterate the results by blob or by page using the iterator returned by the `by_page` function.

## Further reading

[Azure SDK for Python documentation](https://learn.microsoft.com/en-us/python/api/azure-storage-blob/azure.storage.blob.containerclient?view=azure-python#azure-storage-blob-containerclient-find-blobs-by-tags)
