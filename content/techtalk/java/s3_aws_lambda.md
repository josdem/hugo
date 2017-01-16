+++
date = "2017-01-16T13:27:45-06:00"
title = "S3 AWS Lambda"
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology", "aws lambda", "java", "serverless"]

+++

Suppose you want to process a file that is uploaded to a bucket. You can create a Lambda function (BucketFileTransfer) that Amazon S3 can invoke when objects are created. Then, the Lambda function can read the image object from the source bucket and create a copy image target bucket.

In this post I will show you how to create a Java Lambda function in orde to copy a file from one bucket to another. First we need to create a Java basic project this time using lazybones.

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
    logger.log("STRARTING to copy file");

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

To download the code:

```bash
git clone https://github.com/josdem/java-topics.git
cd s3-aws-lambda
```

[Return to the main article](/techtalk/java)
