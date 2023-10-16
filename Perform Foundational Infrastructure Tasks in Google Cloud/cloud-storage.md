# Cloud Storage

### Create a bucket

    gsutil mb gs://bucket-name

### Upload an object to bucket

    gsutil cp object-name gs://bucket-name

### List objects in a bucket

    gsutil ls -r gs://bucket-name

### Download an object from bucket

    gsutil cp -r gs://bucket-name/object-name .

### Copy an object into a folder

    gsutil cp gs://YOUR-BUCKET-NAME/ada.jpg gs://YOUR-BUCKET-NAME/image-folder/

### Make object publicly accessible

    gsutil acl ch -u AllUsers:R gs://YOUR-BUCKET-NAME/ada.jpg

### Remove public access

    gsutil acl ch -d AllUsers gs://YOUR-BUCKET-NAME/ada.jpg

### Delete an object

    gsutil rm gs://YOUR-BUCKET-NAME/ada.jpg


