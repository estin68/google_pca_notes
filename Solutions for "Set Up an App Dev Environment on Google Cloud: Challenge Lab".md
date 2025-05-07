#### Requirements

##### Create the function

Create a Cloud Run Function `memories-thumbnail-generator` that will to create a thumbnail from an image added to the `qwiklabs-gcp-04-b59a2879fec8-bucket` bucket.

Ensure the Cloud Run Function is using the Cloud Run function environment (which is 2nd generation). Ensure the resource is created in the `us-east1` region and `us-east1-d` zone.

1. Create a Cloud Run Function (2nd generation) called `memories-thumbnail-generator` using Node.js 22.

  > **Note:** The Cloud Run Function is required to execute every time an object is created in the bucket created in Task 1. During the process, Cloud Run Function may request permission to enable APIs or request permission to grant roles to service accounts. Please enable each of the required APIs and grant roles as requested.

2. Make sure you set the Entry point (Function to execute) to `memories-thumbnail-generator` and Trigger to `Cloud Storage`.

3. Add the following code to the index.js:

```bash
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

4. Add the following code to the package.json:

```bash
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

### Solution

```bash
mkdir gcf_lab && cd $_
nano index.js
nano package.json
npm install
```

```bash
gcloud functions deploy nodejs-pubsub-function \
  --gen2 \
  --runtime=nodejs20 \
```

```bash
gcloud functions deploy memories-thumbnail-generator \
--gen2 \
--runtime=nodejs22 \
--region=us-east1 \
--source=. \
--entry-point=memories-thumbnail-generator \
--trigger-topic topic-memories-663 \
--stage-bucket qwiklabs-gcp-04-b59a2879fec8-bucket \
--service-account qwiklabs-gcp-04-b59a2879fec8@qwiklabs-gcp-04-b59a2879fec8.iam.gserviceaccount.com \
--allow-unauthenticated
```

```bash
gcloud run deploy memories-thumbnail-generator \
  --image gcr.io/cloud-builders/gcloud-functions-image:nodejs-22 \
  --region us-east1 \
  --source=. \
  --entry-point=memories-thumbnail-generator \
  --trigger-event google.storage.object.finalize \
  --trigger-resource=qwiklabs-gcp-04-b59a2879fec8-bucket \
  --service-account cloudfunctionsa@qwiklabs-gcp-04-b59a2879fec8.iam.gserviceaccount.com \
  --allow-unauthenticated
```

```bash
  gcloud run deploy memories-thumbnail-generator \
  --image gcr.io/cloud-builders/gcloud-functions-image:nodejs-22 \
  --region us-east1 \
  --source=. \
  --set-env-vars=BUCKET_NAME=qwiklabs-gcp-04-b59a2879fec8-bucket
```
