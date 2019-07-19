### cosbench在上put object 比较大的时候出现报错：  

···

com.amazonaws.ResetException: Failed to reset the request input stream; If the request involves an input stream, the maximum stream buffer size can be configured via request.getRequestClie

ntOptions().setReadLimit(int)

        at com.amazonaws.http.AmazonHttpClient.resetRequestInputStream(AmazonHttpClient.java:955)

        at com.amazonaws.http.AmazonHttpClient.executeOneRequest(AmazonHttpClient.java:783)

        at com.amazonaws.http.AmazonHttpClient.executeHelper(AmazonHttpClient.java:607)

        at com.amazonaws.http.AmazonHttpClient.doExecute(AmazonHttpClient.java:376)

        at com.amazonaws.http.AmazonHttpClient.executeWithTimer(AmazonHttpClient.java:338)

        at com.amazonaws.http.AmazonHttpClient.execute(AmazonHttpClient.java:287)

        at com.amazonaws.services.s3.AmazonS3Client.invoke(AmazonS3Client.java:3826)

        at com.amazonaws.services.s3.AmazonS3Client.putObject(AmazonS3Client.java:1405)

        at com.amazonaws.services.s3.AmazonS3Client.putObject(AmazonS3Client.java:1270)

        at com.intel.cosbench.api.S3Stor.S3Storage.createObject(S3Storage.java:167)

        at com.intel.cosbench.driver.operator.Writer.doWrite(Writer.java:98)

        at com.intel.cosbench.driver.operator.Writer.operate(Writer.java:79)

···  
问题原因是：
readLimit默认设置的是 128k  
```java  
        final Integer bufsize = Constants.getS3StreamBufferSize();
        if (bufsize != null) {
            AmazonWebServiceRequest awsreq = request.getOriginalRequest();
            // Note awsreq is never null at this point even if the original
            // request was
            awsreq.getRequestClientOptions()
                  .setReadLimit(bufsize.intValue());
        }   
    public static Integer getS3StreamBufferSize() {
        String s =
            System.getProperty(SDKGlobalConfiguration.DEFAULT_S3_STREAM_BUFFER_SIZE);
        if (s == null)
            return null;
        try {
            return Integer.valueOf(s);
        } catch (Exception e) {
            log.warn("Unable to parse buffer size override from value: " + s);
        }
        return null;
    } 
    public static final String DEFAULT_S3_STREAM_BUFFER_SIZE =
        "com.amazonaws.sdk.s3.defaultStreamBufferSize";
```  
如果com.amazonaws.sdk.s3.defaultStreamBufferSize 有设置，则使用设置的大小，如果没有，这是用默认的128k
所以，这只该值就可以了
