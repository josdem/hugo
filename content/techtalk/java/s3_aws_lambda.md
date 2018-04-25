+++
title = "S3 AWS Lambda"
categories = ["techtalk","code","java"]
tags = ["josdem","techtalks","programming","technology", "aws lambda", "java", "serverless"]
date = "2017-01-16T13:27:45-06:00"
description = "Suppose you want to process a file that is uploaded to a bucket. You can create a Lambda function (BucketFileTransfer) that Amazon S3 can invoke when objects are created. Then, the Lambda function can read the image object from the source bucket and create a copy image target bucket."
+++

Suppose you want to process a file that is uploaded to a bucket. You can create a Lambda function (BucketFileTransfer) that Amazon S3 can invoke when objects are created. Then, the Lambda function can read the image object from the source bucket and create a copy image target bucket.

In this post I will show you how to create a Java Lambda function in order to copy a file (HappyFace.jpg) from one bucket to another. First we need to create a Java basic project this time using lazybones.

```bash
lazybones create java-basic s3-aws-lambda
```

Previous command will create this structure

```bash
<proj>
      |
      +- src
          |
          +- main
          |     |
          |     +- java
          |
          +- test
          |   |
          |   +- java
```

Edit your `build.gradle` file to create a make a Zip task and add lambda AWS dependencies.

```groovy
apply plugin: "java"
apply plugin: "application"

version = '0.0.1'

task buildZip(type: Zip) {
    from compileJava
    from processResources
    into('lib') {
        from configurations.runtime
    }
}

repositories {
  mavenCentral()
}

dependencies {
  compile 'com.amazonaws:aws-lambda-java-core:1.1.0'
  compile 'com.amazonaws:aws-lambda-java-events:1.1.0'
}
```

Then define Java classes under `project-dir/src/main/java/example/`

**BucketFileTransfer**

```java
package example;

import java.io.InputStream;
import com.amazonaws.services.lambda.runtime.events.S3Event;
import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.LambdaLogger;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.amazonaws.services.s3.model.ObjectMetadata;

public class BucketFileTransfer implements RequestHandler<S3Event, Integer> {

  @Override
  public Integer handleRequest(S3Event event, Context context) {
    LambdaLogger logger = context.getLogger();
    logger.log("STARTING to copy file");

    MetadataFileHelper metadataFileHelper = new MetadataFileHelper(event);
    String sourceBucket = metadataFileHelper.getSourceBucketName();
    String sourceKey = metadataFileHelper.getSourceBucketKey();
    String destinationBucket = metadataFileHelper.getDestinationBucketName();;
    String destinationKey = metadataFileHelper.getDestinationBucketKey();

    AWSClient awsClient = new AWSClient(sourceBucket, sourceKey);
    InputStream objectData = awsClient.getS3Object().getObjectContent();
    ObjectMetadata meta = awsClient.getS3Object().getObjectMetadata();
    awsClient.getS3Client().putObject(destinationBucket, destinationKey, objectData, meta);
    return ResultCode.OK;
  }

}
```

Basically in this piece of code we are getting the source bucket, file and metatada. We are getting destination bucket as well, finally we are writing the file in the destination bucket.

**MetadataFileHelper**

```java
package example;

import com.amazonaws.services.lambda.runtime.events.S3Event;
import com.amazonaws.services.s3.event.S3EventNotification.S3Entity;

public class MetadataFileHelper {

  private S3Event event;

  public MetadataFileHelper(S3Event event){
    this.event = event;
  }

  private S3Entity getBucketEntity(){
    return this.event.getRecords().get(0).getS3();
  }

  public String getSourceBucketName() {
    return getBucketEntity().getBucket().getName();
  }

  public String getSourceBucketKey() {
    return getBucketEntity().getObject().getKey();
  }

  public String getDestinationBucketName() {
    return "bucket-s3-destination";
  }

  public String getDestinationBucketKey() {
    return "copied-" + getBucketEntity().getObject().getKey();
  }

}
```

We are using this helper to get source and destination buckets name and keys.

**AWSClient**

```java
package example;

import com.amazonaws.ClientConfiguration;
import com.amazonaws.auth.AWSCredentials;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3Client;
import com.amazonaws.services.s3.model.S3Object;
import com.amazonaws.services.s3.model.GetObjectRequest;

public class AWSClient {

  String sourceBucket;
  String sourceKey;
  AmazonS3 s3Client;

  public AWSClient(String sourceBucket, String sourceKey){
    this.sourceBucket = sourceBucket;
    this.sourceKey = sourceKey;
    AWSCredentials credentials = new BasicAWSCredentials("Access Key ID", "Secret Access Key");
    ClientConfiguration clientConfig = new ClientConfiguration();
    this.s3Client = new AmazonS3Client(credentials, clientConfig);
  }

  public S3Object getS3Object() {
    return getS3Client().getObject(new GetObjectRequest(sourceBucket, sourceKey));
  }

  public AmazonS3 getS3Client(){
    return this.s3Client;
  }
}
```

We are using this client to get AWS connection, Amazon S3 client and Amazon S3 Object.

Use the following gradle command to generate your standalone .zip deployment file

```bash
gradle buildZip
```

Then go to your [AWS console](console.aws.amazon.com) and create a new Lambda AWS function to upload this project.

You can invoke your lambda function installing command line interface [AWS CLI](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)

```bash
aws lambda invoke \
--invocation-type Event \
--function-name s3-aws-lambda \
--region us-west-1 \
--payload file://inputfile.txt \
outputfile.txt
```

Where `inputfile.txt` is:

```json
{
  "Records": [
    {
      "eventVersion": "2.0",
      "eventTime": "1970-01-01T00:00:00.000Z",
      "requestParameters": {
        "sourceIPAddress": "127.0.0.1"
      },
      "s3": {
        "configurationId": "testConfigRule",
        "object": {
          "eTag": "0123456789abcdef0123456789abcdef",
          "sequencer": "0A1B2C3D4E5F678901",
          "key": "HappyFace.jpg",
          "size": 1024
        },
        "bucket": {
          "arn": "arn:aws:s3:::josdem-s3-source",
          "name": "josdem-s3-source",
          "ownerIdentity": {
            "principalId": "EXAMPLE"
          }
        },
        "s3SchemaVersion": "1.0"
      },
      "responseElements": {
        "x-amz-id-2": "EXAMPLE123/5678abcdefghijklambdaisawesome/mnopqrstuvwxyzABCDEFGH",
        "x-amz-request-id": "EXAMPLE123456789"
      },
      "awsRegion": "us-east-1",
      "eventName": "ObjectCreated:Put",
      "userIdentity": {
        "principalId": "EXAMPLE"
      },
      "eventSource": "aws:s3"
    }
  ]
}
```

After invoke command you should see this output:

```json
{
  "StatusCode": 202
}
```

And the HappyFace.jpg image copied.

To download the code:

```bash
git clone https://github.com/josdem/java-topics.git
git fetch
git checkout feature/copy-bucket-file
cd s3-aws-lambda
```

[Return to the main article](/techtalk/java)
