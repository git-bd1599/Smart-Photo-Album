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
      - Use the S3 SDK’s headObject method to retrieve the S3 metadata created at the object’s upload time. Retrieve the x-amz-meta-customLabels metadata field, if applicable,           and create a JSON array (A1) with the labels.
      - Store a JSON object in an ElasticSearch index (“photos”) that references the S3 object from the PUT event (E1) and append string labels to the labels array (A1), one for           each label detected by Rekognition.
       ```
       {
          "objectKey": "my-photo.jpg",
          "bucket": "my-photo-bucket",
          "createdTimestamp": "2018-11-05T12:40:02",
          "labels": [
              "person",
              "dog",
              "ball",
              "park"
            ]
        }
        ```
3. Search
    - Created a Lambda function (LF2) called “search-photos”.
    - Created an Amazon Lex bot to handle search queries.
      - Created one intent named “SearchIntent”.
      - Added training utterances to the intent, such that the bot can pick up both keyword searches (“trees”, “birds”), as well as sentence searches (“show me trees”, “show me         photos with trees and birds in them”).
    - Implemented the Search Lambda function (LF2):
      - Given a search query “q”, disambiguate the query using the Amazon Lex bot.
      - If the Lex disambiguation request yields any keywords (K1, …, Kn), search the “photos” ElasticSearch index for results, and return them accordingly (as per the API               spec).
      - Otherwise, return an empty array of results (as per the API spec).
4. Built the API layer
    - Built an API using API Gateway.
    - The API has two methods:
      - PUT /photos

        Set up the method as an Amazon S3 Proxy . This allows API Gateway to forward your PUT request directly to S3.
        - Use a custom **x-amz-meta-customLabels** HTTP header to include any custom labels the user specifies at upload time.
      - GET /search?q={query text}
      
        Connect this method to the search Lambda function (LF2)
    - Setup an API key for your two API methods
    - Deploy the API.
    - Generate a SDK for the API (SDK1).
5. Frontend
    - Built a simple frontend application that allows users to:
      - Make search requests to the GET /search endpoint
      - Display the results (photos) resulting from the query
      - Upload new photos using the PUT /photos
       - In the upload form, allow the user to specify one or more custom labels, that will be appended to the list of labels detected automatically by Rekognition. These custom          labels should be converted to a comma-separated list and uploaded as part of the S3 object’s metadata using a **x-amz-meta-customLabels** metadata HTTP header.
      
          For instance, if you specify two custom labels at upload time, "Sam" and "Sally", the metadata HTTP header should look like: *x-amz-meta-customLabels: Sam, Sally*
      - Create a S3 bucket for your frontend (B1)
      - Set up the bucket for static website hosting
      - Upload the frontend files to the bucket (B2).
      - Integrate the API Gateway-generated SDK (SDK1) into the frontend, to connect your API.
6. Implement Voice accessibility in the frontend
    - Give the frontend user the choice to use voice rather than text to perform the search.
    - Use Amazon Transcribe on the frontend to transcribe speech to text (STT) in real time , then use the transcribed text to perform the search, using the same API like in the       previous steps.
7. Deployed the code using AWS CodePipeline
    - Define a pipeline (P1) in AWS CodePipeline that builds and deploys the code for/to all your Lambda functions
    - Define a pipeline (P2) in AWS CodePipeline that builds and deploys your frontend code to its corresponding S3 bucket
8. Create a AWS CloudFormation template for the stack
    - Create a CloudFormation template (T1) to represent all the infrastructure resources (ex. Lambdas, ElasticSearch, API Gateway, CodePipeline, etc.) and permissions (IAM           policies, roles, etc.).

At this point you should be able to:
  - Visit your photo album application using the S3 hosted URL.
  - Search photos using natural language via voice and text.
  - See relevant results (ex. If you searched for a cat, you should be able to see
    photos with cats in them) based on what you searched.
  - Upload new photos (with or without custom labels) and see them appear in the
    search results.
    
 
 ![Architecture](https://user-images.githubusercontent.com/61260957/120049287-1c662300-bfdf-11eb-9ff5-595dc21772f1.PNG)

    




      



      


