```
from google.cloud import storage
storage_client = storage.Client()
dstblob_name = ''
srcbucket = storage_client.bucket(bucket_name)
srcblob = srcbucket.blob(blob_name)
dstbucket = storage_client.bucket(bucket_name1)
srcbucket.copy_blob(srcblob, dstbucket, dstblob_name)
# if move
srcblob.delete()
```

## object versioning
- enabled per bucket
- CS retains a nonconcurrent object version each time we replace or delete a live object version
	- nonconcurrent versions retain the name of the object, but are uniquely identified by their generation number
	- NCV only appear in requests that explicitly call for them to be included