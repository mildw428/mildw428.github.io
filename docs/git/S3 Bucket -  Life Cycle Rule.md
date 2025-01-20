---
layout: default
title: S3 Bucket -  Life Cycle Rule
parent: git
---

temp file 업로드시 따로 지우는 로직을 구현해야하는 번거로움이 있습니다.
이는 s3에서 지원하는 Life Cycle Rule을 사용하여 해결할 수 있습니다.

![[Pasted image 20240407214724.png]]


```groovy
// AWS  
implementation platform('software.amazon.awssdk:bom:2.17.102')  
implementation 'software.amazon.awssdk:s3'  
implementation 'software.amazon.awssdk:s3-transfer-manager:2.17.123-PREVIEW'
```

```java
  
public static void tempFileUpload(Bucket bucket, String key, byte[] bytes) {  
    S3Client s3Client = getS3Client(bucket);  
    setLifecyclePolicy(s3Client, bucket.getBucketName());  
  
    ByteBuffer byteBuffer = ByteBuffer.wrap(bytes);  
  
    PutObjectRequest request = PutObjectRequest.builder()  
            .bucket(bucket.getBucketName())  
            .key(key)  
            .contentLength((long) bytes.length)  
            .build();  
  
    s3Client.putObject(request, RequestBody.fromByteBuffer(byteBuffer));  
}  
  
public static void setLifecyclePolicy(S3Client s3Client, String bucketName) {  
  
    List<LifecycleRule> rules = List.of(  
            LifecycleRule.builder()  
                    .id("ExpirationRule")
                    .filter(LifecycleRuleFilter.builder().prefix(key).build())
                    .expiration(LifecycleExpiration.builder().days(1).build())  
                    .status(ExpirationStatus.ENABLED)  
                    .build()  
    );  
  
    PutBucketLifecycleConfigurationRequest request = PutBucketLifecycleConfigurationRequest.builder()  
            .bucket(bucketName)  
            .lifecycleConfiguration(BucketLifecycleConfiguration.builder().rules(rules).build())  
            .build();  
  
    s3Client.putBucketLifecycleConfiguration(request);  
}
```

![[Pasted image 20240407214919.png]]
`.id("ExpirationRule")`id값을 동일하게 설정 할 경우 덮어 씌워버리는점 주의해야합니다.
