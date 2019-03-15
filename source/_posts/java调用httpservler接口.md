---
title: Java调用HttpServleT接口
date: 2018-01-18 18:28:00
tags:
- java
- httpservlet
categories:
- 汇编语言
---


#### 写了一个简单的方法测试调用HttpServlet接口

<!--more-->

```java
public static void testHttpPost() {
	Date date = new Date();

	String requestUrl = "http://ip_address/servlet/~rfffp/nc.rfffp.baseconfig.pub.servlet.CheckContractRefenForErpServlet";
	String userid="10015A1000000009QPTF";
	String dataSource = "nc63pm";
	String billcode = "G1-HO18120009";

	byte[] parameter =
		("{queryCode:\"1\",userid:\""+userid+"\",dataSource:\""+dataSource+"\",pk_group:\"0001A21000000000370V\",contractCode:\""+billcode+"\"}")
		.getBytes();
	String para = new String(parameter);
	try {
		String timestamp = String.valueOf(date.getTime()/1000);
		URL url = new URL(requestUrl);
		URLConnection http_url_connection = url.openConnection();
		HttpURLConnection HttpUrlConnection = (HttpURLConnection) http_url_connection;

		HttpUrlConnection.setDoOutput(true);
		HttpUrlConnection.setDoInput(true);
		HttpUrlConnection.setRequestMethod("POST");//设置请求方式。可以是delete put post get
		HttpUrlConnection.setRequestProperty("Content-Length", String.valueOf(parameter.length));//设置内容的长度
		HttpUrlConnection.setRequestProperty("Content-Type", "application/json;charset=utf-8");//设置编码格式
		HttpUrlConnection.setRequestProperty("accept", "application/json");//设置接收返回参数格式
		HttpUrlConnection.setRequestProperty("timestamp",timestamp);
		HttpUrlConnection.setUseCaches(false);

		BufferedOutputStream output_stream = new BufferedOutputStream(HttpUrlConnection.getOutputStream());
		output_stream.write(parameter);
		output_stream.flush();
		output_stream.close();
		output_stream = null;

		InputStreamReader input_stream_reader = new InputStreamReader(HttpUrlConnection.getInputStream(), "utf-8");
		BufferedReader buffered_reader = new BufferedReader(input_stream_reader);

		String line;
		StringBuffer buffer = new StringBuffer();
		while ((line = buffered_reader.readLine()) != null) {
			buffer.append(line);
		}
		System.out.println(buffer.toString());
		System.out.println(para);
	} catch (MalformedURLException e) {
		e.printStackTrace();
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```
