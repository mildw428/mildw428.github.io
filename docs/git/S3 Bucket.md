---
layout: default
title: S3 Bucket
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/core/src/main/java/com/example/core/s3/S3Utils.java

S3 파일 업로드, 다운로드 

```java
//Upload
    public static boolean upload(Bucket bucket, String key, byte[] bytes) {
        try (S3TransferManager s3TransferManager = getS3TransferManager(bucket)) {
            PutObjectRequest putObjectRequest = PutObjectRequest.builder().bucket(bucket.getBucketName()).key(key)
                    .serverSideEncryption(ServerSideEncryption.AES256).build();
            UploadRequest uploadRequest = UploadRequest.builder().putObjectRequest(putObjectRequest)
                    .requestBody(AsyncRequestBody.fromBytes(bytes)).build();

            Upload upload = s3TransferManager.upload(uploadRequest);
            CompletedUpload completedUpload = upload.completionFuture().join();
            PutObjectResponse putObjectResponse = completedUpload.response();

            return putObjectResponse != null;
        } catch (Exception e) {
            throw new RuntimeException();
        }
    }
```

```java
//Download
    public static byte[] download(Bucket bucket, String key) {
        try (S3TransferManager s3TransferManager = getS3TransferManager(bucket)) {
            GetObjectRequest getObjectRequest = GetObjectRequest.builder().bucket(bucket.getBucketName()).key(key).build();
            DownloadRequest<ResponseBytes<GetObjectResponse>> downloadRequest = DownloadRequest.builder()
                    .getObjectRequest(getObjectRequest).responseTransformer(AsyncResponseTransformer.toBytes()).build();

            Download<ResponseBytes<GetObjectResponse>> download = s3TransferManager.download(downloadRequest);
            CompletedDownload<ResponseBytes<GetObjectResponse>> completedDownload = download.completionFuture().join();
            ResponseBytes<GetObjectResponse> responseBytes = completedDownload.result();

            return responseBytes.asByteArray();
        } catch (Exception exception) {
            throw new RuntimeException();
        }
    }
```

```java
    private static S3Client getS3Client(Bucket bucket) {
        S3ClientBuilder s3ClientBuilder = S3Client.builder().region(AwsUtils.getAwsRegion());

        if (bucket.getAccessKey() != null && bucket.getSecretKey() != null) {
            s3ClientBuilder.credentialsProvider(StaticCredentialsProvider.create(
                    AwsBasicCredentials.create(bucket.getAccessKey(), bucket.getSecretKey())));
        }

        return s3ClientBuilder.build();
    }

    private static S3TransferManager getS3TransferManager(Bucket bucket) {
        S3ClientConfiguration s3ClientConfiguration = getS3ClientConfiguration(bucket);
        return S3TransferManager.builder().s3ClientConfiguration(s3ClientConfiguration).build();
    }

    private static S3ClientConfiguration getS3ClientConfiguration(Bucket bucket) {
        S3ClientConfiguration.Builder builder = S3ClientConfiguration.builder().region(AwsUtils.getAwsRegion());

        if (bucket.getAccessKey() != null && bucket.getSecretKey() != null) {
            builder.credentialsProvider(StaticCredentialsProvider.create(
                    AwsBasicCredentials.create(bucket.getAccessKey(), bucket.getSecretKey())));
        }

        return builder.build();
    }
```

