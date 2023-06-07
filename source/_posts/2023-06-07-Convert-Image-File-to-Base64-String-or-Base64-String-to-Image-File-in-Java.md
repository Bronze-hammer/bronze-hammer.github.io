---
title: Convert Image File to Base64 String or Base64 String to Image File in Java
toc: true
date: 2023-06-07 18:11:27
tags:
categories:
	- 代码人生
---


## Add Maven Dependency

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

## Image File Convert Base64 String


```java
byte[] fileContent = FileUtils.readFileToByteArray(new File(filePath));
String encodedString = Base64.getEncoder().encodeToString(fileContent);

```


## Base64 String Convert Image File

```java
byte[] decodedBytes = Base64.getDecoder().decode(encodedString);
FileUtils.writeByteArrayToFile(new File(outputFileName), decodedBytes);
```

