# Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab

> Launch the lab [here](https://google.qwiklabs.com/quests/118)

In this lab there is no command line instructions, you have to just use the **Google Cloud Platform (GCP)** for completing this lab. A walkthrough of each task is available below: 

## Your challenge

You are now asked to help a newly formed development team with some of their initial work on a new project around storing and organizing photographs, called memories. You have been asked to assist the memories team with initial configuration for their application development environment; you receive the following request to complete the following tasks:

- Create a bucket for storing the photographs.
- Create a Pub/Sub topic that will be used by a Cloud Function you create.
- Create a Cloud Function.
- Remove the previous cloud engineer’s access from the memories project.

Some Jooli Inc. standards you should follow:

- Create all resources in the **us-east1** region and **us-east1-b** zone, unless otherwise directed.
- Use the projectdatd VPCs.
- Naming is normally *team-resource*, e.g. an instance could be named **kraken-webserver1**
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share; unless directed, use **f1-micro** for small Linux VMs and **n1-standard-1** for Windows or other applications such as Kubernetes nodes.

## Solving tasks

### Task 1: Create a bucket

1. Navigation menu > **Cloud Storage** > Browser > Create Bucket
2. Name your bucket > Enter **Given Bucket Name** > Continue
3. Choose where to store your data > **Region:** us-east1 > Continue
4. Use default for the remaining 
5. Create

### Task 2: Create a Pub/Sub topic

1. Navigation menu > **Pub/Sub** > Topics
2. Create Topic > Enter **Given Topic Name:**  > Create Topic

### Task 3: Create the thumbnail Cloud Function

1. Navigation menu > **Cloud Functions** > Create Function

2. Use the following config:

   **Name:** Enter the Name given In the Side Bar
   **Region:** us-east1
   **Trigger:** Cloud Storage
   **Event type:** Finalize/Create
   **Bucket:** BROWSE > Select the qwiklabs bucket
   Now **Save** it

3. Remaining default > Next

4. **Runtime:** >=Node.js 10
   **Entry point:** thumbnails
5. Add the following code in index.js * Change **tpoicname** in <>
```yaml
/* globals exports, require */
//jshint strict: false
//jshint esversion: 6
"use strict";
const crc32 = require("fast-crc32c");
const { Storage } = require('@google-cloud/storage');
const gcs = new Storage();
const { PubSub } = require('@google-cloud/pubsub');
const imagemagick = require("imagemagick-stream");
exports.thumbnail = (event, context) => {
  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64"
  const bucket = gcs.bucket(bucketName);
  const topicName = "<REPLACE_WITH_YOUR_TOPIC ID>";
  const pubsub = new PubSub();
  if ( fileName.search("64x64_thumbnail") == -1 ){
    // doesn't have a thumbnail, get the filename extension
    var filename_split = fileName.split('.');
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length );
    if (filename_ext.toLowerCase() == 'png' || filename_ext.toLowerCase() == 'jpg'){
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      let newFilename = filename_without_ext + size + '_thumbnail.' + filename_ext;
      let gcsNewObject = bucket.file(newFilename);
      let srcStream = gcsObject.createReadStream();
      let dstStream = gcsNewObject.createWriteStream();
      let resize = imagemagick().resize(size).quality(90);
      srcStream.pipe(resize).pipe(dstStream);
      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
              // set the content-type
              gcsNewObject.setMetadata(
              {
                contentType: 'image/'+ filename_ext.toLowerCase()
              }, function(err, apiResponse) {});
              pubsub
                .topic(topicName)
                .publisher()
                .publish(Buffer.from(newFilename))
                .then(messageId => {
                  console.log(`Message ${messageId} published.`);
                })
                .catch(err => {
                  console.error('ERROR:', err);
                });
          });
      });
    }
    else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  }
  else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
};
```
6. Add the following code in package.json
```yaml
{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/pubsub": "^2.0.0",
    "@google-cloud/storage": "^5.0.0",
    "fast-crc32c": "1.0.4",
    "imagemagick-stream": "4.1.1"
  },
  "devDependencies": {},
  "engines": {
    "node": ">=4.3.2"
  }
}
```
7. Download the image from URL
8. Navigation menu > **Cloud Storage** > Browser > Select your bucket > Upload files
9. Refresh bucket

### Task 4: Remove the previous cloud engineer

1. Navigation menu > **IAM & Admin** > IAM
2. Search for the "**Username 2**" > Edit > Delete Role
