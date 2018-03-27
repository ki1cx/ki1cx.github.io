---
layout: single
title: "Video Chunked Upload in Javascript"
description: Explore video chunked upload in javascript to upload videos of any size to Facebook marketing api. This serverless solution only uses the browser native apis to read the file from user input and reads blobs of data as requested by the FB api. This is also called the resumable upload.
date: 2018-03-09 07:46:14 -0000
categories: facebook api javascript
---

A scalable file uploading solution requires a chunk upload strategy, since large files cannot be held in memory at some point. I've been working on a project that requires uploading video files directly to Facebook servers without the backend being involved. 

Here is the link [fb-video-uploader](https://www.npmjs.com/package/fb-video-uploader){:target="_blank"} to the module I created, which has no dependencies except babel-runtime to support async/await.

#### There are three stages of upload. 

* start - request with file size -> returns the chunk's start/end offset and the upload_session_id

	Request Headers
	
	```
	:authority: graph-video.facebook.com
	:method: POST
	:path: /v2.11/<ad account id>/advideos
	:scheme: https
	accept: application/json
	accept-encoding: gzip, deflate, br
	accept-language: en-US,en;q=0.9,bs;q=0.8
	content-length: 533
	content-type: multipart/form-data; boundary=----WebKitFormBoundaryAuH710A4wAnvEAnL
	origin: http://localhost:3000
	referer: http://localhost:3000/
	user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.146 Safari/537.36	
	```
	
	Request Payload
	
	```
	------WebKitFormBoundaryAuH710A4wAnvEAnL
	Content-Disposition: form-data; name="access_token"
	
	<access token>
	------WebKitFormBoundaryAuH710A4wAnvEAnL
	Content-Disposition: form-data; name="upload_phase"
	
	start
	------WebKitFormBoundaryAuH710A4wAnvEAnL
	Content-Disposition: form-data; name="file_size"
	
	369763
	------WebKitFormBoundaryAuH710A4wAnvEAnL--
	```
	
	Response	

	```javascript
	{
	  upload_session_id: "10157823606217837",
	  start_offset: "0",
	  end_offset: "369763",
	  video_id: "125123125125"
	}
	```

* transfer - request with upload_session_id, starting index and file blob of chunk size -> returns the next chunk's start/end offset

	Request Headers
	
	```
	:authority: graph-video.facebook.com
	:method: POST
	:path: /v2.11/<ad account id>/advideos
	:scheme: https
	accept: application/json
	accept-encoding: gzip, deflate, br
	accept-language: en-US,en;q=0.9,bs;q=0.8
	content-length: 370578
	content-type: multipart/form-data; boundary=----WebKitFormBoundaryjWtlmACzRHxI1JLV
	origin: http://localhost:3000
	referer: http://localhost:3000/
	user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.146 Safari/537.36
	```
	
	Request Payload
	
	```
	------WebKitFormBoundaryjWtlmACzRHxI1JLV
	Content-Disposition: form-data; name="access_token"
	
	<access token>
	------WebKitFormBoundaryjWtlmACzRHxI1JLV
	Content-Disposition: form-data; name="upload_phase"
	
	transfer
	------WebKitFormBoundaryjWtlmACzRHxI1JLV
	Content-Disposition: form-data; name="upload_session_id"
	
	10157823606217837
	------WebKitFormBoundaryjWtlmACzRHxI1JLV
	Content-Disposition: form-data; name="start_offset"
	
	0
	------WebKitFormBoundaryjWtlmACzRHxI1JLV
	Content-Disposition: form-data; name="video_file_chunk"; filename="blob"
	Content-Type: application/octet-stream
	
	
	------WebKitFormBoundaryjWtlmACzRHxI1JLV--
	```

	Response
	

	```javascript
	{
	  start_offset: "369763",
	  end_offset: "369763"
	}
	```

* finish - request with upload_session_id and video title -> returns completed status

	Request Headers
	
	```
	:authority: graph-video.facebook.com
	:method: POST
	:path: /v2.11/<ad account id>/advideos
	:scheme: https
	accept: application/json
	accept-encoding: gzip, deflate, br
	accept-language: en-US,en;q=0.9,bs;q=0.8
	content-length: 553
	content-type: multipart/form-data; boundary=----WebKitFormBoundary0tqCdIckXVm1JTyG
	origin: http://localhost:3000
	referer: http://localhost:3000/
	user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.146 Safari/537.36
	```
	Request Payload
	
	```
	------WebKitFormBoundary0tqCdIckXVm1JTyG
	Content-Disposition: form-data; name="access_token"
	
	<access token>
	------WebKitFormBoundary0tqCdIckXVm1JTyG
	Content-Disposition: form-data; name="upload_phase"
	
	finish
	------WebKitFormBoundary0tqCdIckXVm1JTyG
	Content-Disposition: form-data; name="upload_session_id"
	
	10157823606217837
	------WebKitFormBoundary0tqCdIckXVm1JTyG--
	```
	Response

	```javascript
	{
	  success: true
	}
	```

#### Notes

Requests are made to 

	https://graph-video.facebook.com/v2.11/<ad account id>/advideos
	
Using the the following content type (this is automatically set by XMLHttpRequest.

	content-type: multipart/form-data

Get the File object from user input.

	<input type="file" onchange="handleInputChange">

Get the file [size](https://developer.mozilla.org/en-US/docs/Web/API/File/size){:target="_blank"}.

	file.size

Get the chunks using the [File.slice](https://developer.mozilla.org/en-US/docs/Web/API/File){:target="_blank"} method.

	file.slice(start_offset, end_offset + 1);

By using the FormData to include the necessary data required for the requests, videos can uploaded easily as a serverless solution. 

Certain browsers do not support FormData.set/delete methods, so create a new instance of FormData for each request and use the *append* method.

	const formData = new FormData();
	formData.append(key, value);

Don't forget to include the access_token in the requests.

