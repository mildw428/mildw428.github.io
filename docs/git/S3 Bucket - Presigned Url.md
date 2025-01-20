---
layout: default
title: S3 Bucket - Presigned Url
parent: git
---

```java
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import software.amazon.awssdk.services.s3.presigner.S3Presigner;  
  
@Configuration  
public class S3FileConfig {  
    @Bean  
    public S3Presigner getS3Presigner() {  
        return S3Presigner.create();  
    }  
  
}
```
Presinger 생성에는 비용이 많이 들기 때문에 @Bean으로 등록 후 사용하는것이 좋다.

```java
public static String getPresignedUrl(S3Presigner s3Presigner, Bucket bucket, String key, Duration duration) {  
    GetObjectRequest getObjectRequest = GetObjectRequest.builder()  
            .bucket(bucket.getBucketName())  
            .key(key)  
            .build();  
  
    GetObjectPresignRequest getObjectPresignRequest = GetObjectPresignRequest.builder()  
            .signatureDuration(duration)  
            .getObjectRequest(getObjectRequest)  
            .build();  
  
    PresignedGetObjectRequest presignedGetObjectRequest = s3Presigner.presignGetObject(getObjectPresignRequest);  
  
    return presignedGetObjectRequest.url().toString();  
}
```


