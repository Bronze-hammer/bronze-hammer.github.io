---
title: Minio文件服务器：SpringBoot实现文件上传
toc: true
date: 2024-10-27 12:40:06
tags:
 - Spring
 - minio
categories:
 - 文件服务器
---

在Minio文件服务器部署成功后(参考上篇文章《Minio文件服务器：安装]》)接下来我们通过SpringBoot框架写一个接口，来实现文件的上传功能：文件通过SpringBoot接口，上传到Minio文件服务器。并且，如果上传的文件是图片类型，也要实现能够预览上传后的图片。

<!-- more -->

## 1 完成后的项目结构

![微信截图 20241027125354](https://img.picgo.net/2024/10/27/_2024102712535402142b0eb619cc1e.jpeg)

## 2 代码

### 2.1 MinioserverApplication.java

```java
package com.example.minioserver;

import com.example.minioserver.config.MinioProperties;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;

@EnableConfigurationProperties(value = {MinioProperties.class})
@SpringBootApplication
public class MinioserverApplication {

	public static void main(String[] args) {
		SpringApplication.run(MinioserverApplication.class, args);
	}

}

```

### 2.2 MinioProperties.java

```java
package com.example.minioserver.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;

/**
 * @author: xbronze
 * @date: 2024-10-26 17:04
 * @description: TODO
 */
@Data
@ConfigurationProperties(prefix="minio") //读取节点
public class MinioProperties {

    private String endpointUrl;
    private String accessKey;
    private String secreKey;
    private String bucketName;

}
```

### 2.3 ViewContentType.java

```java
package com.example.minioserver.enums;

import cn.hutool.core.util.StrUtil;

public enum ViewContentType {

    DEFAULT("default","application/octet-stream"),
    JPG("jpg", "image/jpeg"),
    TIFF("tiff", "image/tiff"),
    GIF("gif", "image/gif"),
    JFIF("jfif", "image/jpeg"),
    PNG("png", "image/png"),
    TIF("tif", "image/tiff"),
    ICO("ico", "image/x-icon"),
    JPEG("jpeg", "image/jpeg"),
    WBMP("wbmp", "image/vnd.wap.wbmp"),
    FAX("fax", "image/fax"),
    NET("net", "image/pnetvue"),
    JPE("jpe", "image/jpeg"),
    RP("rp", "image/vnd.rn-realpix");

    private String prefix;

    private String type;

    public static String getContentType(String prefix){
        if(StrUtil.isEmpty(prefix)){
            return DEFAULT.getType();
        }
        prefix = prefix.substring(prefix.lastIndexOf(".") + 1);
        for (ViewContentType value : ViewContentType.values()) {
            if(prefix.equalsIgnoreCase(value.getPrefix())){
                return value.getType();
            }
        }
        return DEFAULT.getType();
    }

    ViewContentType(String prefix, String type) {
        this.prefix = prefix;
        this.type = type;
    }

    public String getPrefix() {
        return prefix;
    }

    public String getType() {
        return type;
    }

}

```

### 2.4 FileUploadService.java

```java
package com.example.minioserver.service;

import org.springframework.web.multipart.MultipartFile;

/**
 * @author: xbronze
 * @date: 2024-10-26 16:59
 * @description: TODO
 */
public interface FileUploadService {

    public String fileUpload(MultipartFile multipartFile);

}

```

### 2.5 FileUploadServiceImpl.java

```java
package com.example.minioserver.service.impl;

import cn.hutool.core.date.DateUtil;
import cn.hutool.core.lang.UUID;
import com.example.minioserver.config.MinioProperties;
import com.example.minioserver.enums.ViewContentType;
import com.example.minioserver.service.FileUploadService;
import io.minio.BucketExistsArgs;
import io.minio.MakeBucketArgs;
import io.minio.MinioClient;
import io.minio.PutObjectArgs;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.util.Date;

/**
 * @author: xbronze
 * @date: 2024-10-26 17:00
 * @description: TODO
 */
@Service
public class FileUploadServiceImpl implements FileUploadService {

    @Autowired
    private MinioProperties minioProperties;

    @Override
    public String fileUpload(MultipartFile multipartFile) {
        try {
            // 创建一个Minio的客户端对象
            MinioClient minioClient = MinioClient.builder()
                    .endpoint(minioProperties.getEndpointUrl())
                    .credentials(minioProperties.getAccessKey(), minioProperties.getSecreKey())
                    .build();

            // 判断桶是否存在
            boolean found = minioClient.bucketExists(BucketExistsArgs
            	.builder()
            	.bucket(minioProperties.getBucketName())
            	.build());
            if (!found) {       // 如果不存在，那么此时就创建一个新的桶
                minioClient.makeBucket(MakeBucketArgs
                	.builder()
                	.bucket(minioProperties.getBucketName())
                	.build());
            } else {  // 如果存在打印信息
                System.out.println("Bucket " 
                	+ minioProperties.getBucketName() 
                	+ " already exists.");
            }

            // 设置存储对象名称
            String dateDir = DateUtil.format(new Date(), "yyyyMMdd");
            String uuid = UUID.randomUUID().toString().replace("-", "");
            //20230801/443e1e772bef482c95be28704bec58a901.jpg
            String fileName = dateDir+"/"+uuid+multipartFile.getOriginalFilename();
            System.out.println(fileName);

            PutObjectArgs putObjectArgs = PutObjectArgs.builder()
                    .bucket(minioProperties.getBucketName())
                    .stream(multipartFile.getInputStream(), multipartFile.getSize(), -1)
                    .object(fileName)
                    // 上传时指定对应对ContentType
                    // Content-Type 用于向接收方说明传输资源的媒体类型，
                    // 从而让浏览器用指定码表去解码。
                    .contentType(ViewContentType.getContentType(fileName))
                    .build();
            minioClient.putObject(putObjectArgs) ;

            return minioProperties.getEndpointUrl() 
            + "/" + minioProperties.getBucketName() 
            + "/" + fileName ;

        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

}

```

### 2.6 FileUploadController.java

```java
package com.example.minioserver.controller;

import com.example.minioserver.service.FileUploadService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.util.HashMap;
import java.util.Map;

/**
 * @author: xbronze
 * @date: 2024-10-26 16:59
 * @description: TODO
 */
@RestController
@RequestMapping("/file")
public class FileUploadController {

    @Autowired
    private FileUploadService fileUploadService ;

    @PostMapping(value = "/upload")
    public Map<String,String> fileUploadService(
    		@RequestParam(value = "file") MultipartFile multipartFile) {
        Map<String, String> map = new HashMap<>();
        String fileUrl = fileUploadService.fileUpload(multipartFile);
        map.put("fileurl",fileUrl);
        return map;
    }

}

```


### 2.7 application.yml

```xml
spring:
  application:
    name: minioserver
server:
  port: 8091

# 部署的minio服务
minio:
  endpointUrl: http://192.168.72.135:9100
  accessKey: admin
  secreKey: admin123
  bucketName: test
```

### 2.8 pom.xml

这里只展示引入的依赖

```xml
<dependencies>
	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-web</artifactId>
	</dependency>

	<dependency>
	    <groupId>org.projectlombok</groupId>
	    <artifactId>lombok</artifactId>
	    <optional>true</optional>
	</dependency>
	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-test</artifactId>
	    <scope>test</scope>
	</dependency>

	<!-- https://mvnrepository.com/artifact/io.minio/minio -->
	<dependency>
		<groupId>io.minio</groupId>
		<artifactId>minio</artifactId>
		<version>8.5.10</version>
	</dependency>

	<!-- https://mvnrepository.com/artifact/cn.hutool/hutool-all -->
	<dependency>
	    <groupId>cn.hutool</groupId>
	    <artifactId>hutool-all</artifactId>
	    <version>5.8.32</version>
	</dependency>

</dependencies>
```


## 3 运行

访问文件上传接口，选择一张图片上传，接口请求成功后，会返回一个url链接。

![](https://img.picgo.net/2024/10/27/_2024102713073412f0d8c029a00a84.jpeg)

通过链接可预览上传的图片。

![](https://img.picgo.net/2024/10/27/_202410271308198d9786595967d987.jpeg)