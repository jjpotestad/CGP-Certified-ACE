# Cloud Storage: Qwik Start - CLI/SDK

## Setup and requirements

#### (Optional) You can list the active account name with this command:
```bash
gcloud auth list
```

#### (Optional) You can list the project ID with this command:
```bash
gcloud config list project
```

#### In Cloud Shell, set the default region:
```bash
gcloud config set compute/region <REGION>
```

## Task 1. Create a bucket

#### In Cloud Shell, create a bucket
```bash
gcloud storage buckets create gs://<YOUR-BUCKET-NAME>
```

#### Set the default zone:
```bash
gcloud config set compute/zone <ZONE>
```

## Task 2. Upload an object into your bucket

#### To download this image (ada.jpg) into your bucket, enter this command into Cloud Shell
```bash
curl https://upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Ada_Lovelace_portrait.jpg/800px-Ada_Lovelace_portrait.jpg --output ada.jpg
```

#### Use the <b>gcloud storage cp</b> command to upload the image from the location where you saved it to the bucket you created
```bash
gcloud storage cp ada.jpg gs://YOUR-BUCKET-NAME
```

#### Now remove the downloaded image:
```bash
rm ada.jpg
```

## Task 3. Download an object from your bucket

#### Use the <b>gcloud storage cp</b> command to download the image you stored in your bucket to Cloud Shell:
```bash
gcloud storage cp -r gs://YOUR-BUCKET-NAME/ada.jpg .
```

## Task 4. Copy an object to a folder in the bucket

#### Use the <b>gcloud storage cp</b> command to create a folder called image-folder and copy the image (ada.jpg) into it:
```bash
gcloud storage cp gs://YOUR-BUCKET-NAME/ada.jpg gs://YOUR-BUCKET-NAME/image-folder/
```

## Task 5. List contents of a bucket or folder

#### Use the <b>gcloud storage ls</b> command to list the contents of the bucket:

```bash
gcloud storage ls gs://YOUR-BUCKET-NAME
```

## Task 6. List details for an object

#### Use the <b>gcloud storage ls</b> command, with the <b>-l</b> flag to get some details about the image file you uploaded to your bucket:

```bash
gcloud storage ls -l gs://YOUR-BUCKET-NAME/ada.jpg
```

## Task 7. Make your object publicly accessible

#### Use the <b>gsutil acl ch</b> command to grant all users read permission for the object stored in your bucket:

```bash
gsutil acl ch -u AllUsers:R gs://YOUR-BUCKET-NAME/ada.jpg
```

## Task 8. Remove public access

#### To remove this permission, use the command:

```bash
gsutil acl ch -d AllUsers gs://YOUR-BUCKET-NAME/ada.jpg
```

#### Use the <b>gcloud storage rm</b> command to delete an object - the image file in your bucket:
```bash
gcloud storage rm gs://YOUR-BUCKET-NAME/ada.jpg
```

## Congratulations!
