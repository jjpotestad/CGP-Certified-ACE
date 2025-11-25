# Set Up an App Dev Environment on Google Cloud: Challenge Lab

## Setup and requirements

#### (Optional) You can list the active account name with this command:
```bash
gcloud auth list
```

#### (Optional) You can list the project ID with this command:
```bash
gcloud config list project
```

## Challenge scenario

#### In Cloud Shell, set the default region:
```bash
gcloud config set compute/region us-east1 
```

#### Set the default zone:
```bash
gcloud config set compute/zone us-east1-c 
```

## Task 1. Create a bucket

#### In Cloud Shell, create a bucket
```bash
gcloud storage buckets create gs://qwiklabs-gcp-03-1ee18dc7dad8-bucket
--region=us-east1
```

## Task 2. Create Pub/Sub topics

#### Run the following command to create a topic:
```bash
gcloud pubsub topics create topic-memories-662
```


## Task 3. Create the thumbnail Cloud Run Function
#### Ensure the Cloud Run Function is using the Cloud Run function environment (which is 2nd generation).Ensure the resource is created in the REGION region and ZONE zone. Create a Cloud Run Function (2nd generation) called Cloud Run Function Name using Node.js 22.

#### 3.1 Create a directory for the function code
```bash
mkdir gcf_thumbnail && cd $_
```

#### 3.2 Create and open index.js to edit:
```bash
nano index.js
```

#### 3.3 Copy and save the following into the index.js file:
```javascript
const functions = require('@google-cloud/functions-framework');
const { Storage } = require('@google-cloud/storage');
const { PubSub } = require('@google-cloud/pubsub');
const sharp = require('sharp');

functions.cloudEvent('', async cloudEvent => {
  const event = cloudEvent.data;

  console.log(`Event: ${JSON.stringify(event)}`);
  console.log(`Hello ${event.bucket}`);

  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64";
  const bucket = new Storage().bucket(bucketName);
  const topicName = "";
  const pubsub = new PubSub();

  if (fileName.search("64x64_thumbnail") === -1) {
    // doesn't have a thumbnail, get the filename extension
    const filename_split = fileName.split('.');
    const filename_ext = filename_split[filename_split.length - 1].toLowerCase();
    const filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length - 1); // fix sub string to remove the dot

    if (filename_ext === 'png' || filename_ext === 'jpg' || filename_ext === 'jpeg') {
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      const newFilename = `${filename_without_ext}_64x64_thumbnail.${filename_ext}`;
      const gcsNewObject = bucket.file(newFilename);

      try {
        const [buffer] = await gcsObject.download();
        const resizedBuffer = await sharp(buffer)
          .resize(64, 64, {
            fit: 'inside',
            withoutEnlargement: true,
          })
          .toFormat(filename_ext)
          .toBuffer();

        await gcsNewObject.save(resizedBuffer, {
          metadata: {
            contentType: `image/${filename_ext}`,
          },
        });

        console.log(`Success: ${fileName} → ${newFilename}`);

        await pubsub
          .topic(topicName)
          .publishMessage({ data: Buffer.from(newFilename) });

        console.log(`Message published to ${topicName}`);
      } catch (err) {
        console.error(`Error: ${err}`);
      }
    } else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  } else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
});
```

#### 3.4 Create and open package.json to edit:
```bash
nano package.json
```

#### 3.5 Copy and save the following into the package.json file:
```json
{
 "name": "thumbnails",
 "version": "1.0.0",
 "description": "Create Thumbnail of uploaded image",
 "scripts": {
   "start": "node index.js"
 },
 "dependencies": {
   "@google-cloud/functions-framework": "^3.0.0",
   "@google-cloud/pubsub": "^2.0.0",
   "@google-cloud/storage": "^6.11.0",
   "sharp": "^0.32.1"
 },
 "devDependencies": {},
 "engines": {
   "node": ">=4.3.2"
 }
}

```

#### 3.6 Install the package dependencies
```bash
npm install
```

#### 3.7 Deploy your function the memories-thumbnail-generator function to a pub/sub topic named cf-demo

```bash
gcloud functions deploy memories-thumbnail-generator \
  --gen2 \
  --runtime=nodejs20 \
  --region=us-east1 \
  --source=. \
  --entry-point=memories-thumbnail-generator \
  --trigger-event=google.cloud.storage.object.v1.finalized \
  --trigger-resource=qwiklabs-gcp-03-1ee18dc7dad8-bucket \
  --trigger-location=us-east1 \
  --service-account 1075621054860-compute@developer.gserviceaccount.com
```

#### 3.8 Verify the status of the function:
```bash
gcloud functions describe memories-thumbnail-generator \
  --region=us-east1
```

## Task 3. Test the function
#### Upload a PNG or JPG image of your choice to the Bucket Name bucket.
#### Note: Alternatively, download this image https://storage.googleapis.com/cloud-training/gsp315/map.jpg to your machine. Then, upload it to the bucket.

## Task 4. Remove the previous cloud engineer
#### You will see that there are two users defined in the project.
One is your account (Username 1 with the role of Owner).
The other is the previous cloud engineer (Username 2 with the role of Viewer).

## Congratulations!