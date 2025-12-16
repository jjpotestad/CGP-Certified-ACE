# Cloud Run Functions: Qwik Start - Command Line

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

## Task 1. Create a function
#### 1.1 Create a directory for the function code
```bash
mkdir gcf_hello_world && cd $_
```

#### 1.2 Create and open index.js to edit:
```bash
nano index.js
```

#### 1.3 Copy and save the following into the index.js file:
```javascript
const functions = require('@google-cloud/functions-framework');

// Register a CloudEvent callback with the Functions Framework that will
// be executed when the Pub/Sub trigger topic receives a message.
functions.cloudEvent('helloPubSub', cloudEvent => {
  // The Pub/Sub message is passed as the CloudEvent's data payload.
  const base64name = cloudEvent.data.message.data;

  const name = base64name
    ? Buffer.from(base64name, 'base64').toString()
    : 'World';

  console.log(`Hello, ${name}!`);
});
```

#### 1.4 Create and open package.json to edit:
```bash
nano package.json
```

#### 1.5 Copy and save the following into the package.json file:
```json
{
  "name": "gcf_hello_world",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "@google-cloud/functions-framework": "^3.0.0"
  }
}
```

#### 1.6 Install the package dependencies
```bash
npm install
```

## Task 2. Deploy your function

#### 2.1 Deploy the nodejs-pubsub-function function to a pub/sub topic named cf-demo

```bash
gcloud functions deploy nodejs-pubsub-function \
  --gen2 \
  --runtime=nodejs20 \
  --region=<REGION> \
  --source=. \
  --entry-point=<functionEntryPoint> \
  --trigger-topic cf-demo \
  --stage-bucket YOUR-BUCKET-NAME \
  --service-account cloudfunctionsa@PROJECT_ID.iam.gserviceaccount.com \
  --allow-unauthenticated
```

#### 2.2 Verify the status of the function:
```bash
gcloud functions describe nodejs-pubsub-function \
  --region=<REGION> 
```

## Task 3. Test the function

#### After you deploy the function and know that it's active, test that the function writes a message to the cloud log after detecting an event. Invoke the PubSub with some data:
```bash
gcloud pubsub topics publish cf-demo --message="Cloud Function Gen2"
```

## Task 4. View logs

#### Check the logs to see your messages in the log history:
```bash
gcloud functions logs read nodejs-pubsub-function \
  --region=<REGION> 
```

## Congratulations!
