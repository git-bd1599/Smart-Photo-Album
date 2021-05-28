# Smart Photo Album

Implemented a photo album web application, that can be searched using natural language through both text and voice.  We used Lex, ElasticSearch, and Rekognition to create an intelligent search layer to query photos for people, objects, actions, landmarks and more.

### Outline: 
1. Launched an ElasticSearch instance
2. Uploaded & indexed photos
  - Created a S3 bucket (B2) to store the photos.
  - Create a Lambda function (LF1) called “index-photos”.
  - Set up a PUT event trigger on the photos S3 bucket (B2), such that whenever a photo gets uploaded to the bucket, it triggers the Lambda function (LF1) to index it.
  - Implement the indexing Lambda function (LF1):
    - Given a S3 PUT event (E1) detect labels in the image, using Rekognition (“detectLabels” method).
    - Use the S3 SDK’s headObject method to retrieve the S3 metadata created at the object’s upload time. Retrieve the x-amz-meta-customLabels metadata field, if applicable, and         create a JSON array (A1) with the labels.
    - Store a JSON object in an ElasticSearch index (“photos”) that references the S3 object from the PUT event (E1) and append string labels to the labels array (A1), one for           each label detected by Rekognition.
        ```
       {
          “objectKey”: “my-photo.jpg”,
          “bucket”: “my-photo-bucket”,
          “createdTimestamp”: “2018-11-05T12:40:02”,
          “labels”: [
              “person”,
              “dog”,
              “ball”,
              “park”
            ]
        }
        ```

