# ass v4.0.3

Aptoma Smooth Storage

- [File](#file)
	- [Delete File](#delete-file)
	- [Get File Details](#get-file-details)
	- [Get File Content](#get-file-content)
	- [Upload / Modify File](#upload-/-modify-file)
	- [Upload File](#upload-file)
	- [Search Files](#search-files)
	
- [Image](#image)
	- [Delete File](#delete-file)
	- [Get Image Details](#get-image-details)
	- [Live Transform](#live-transform)
	- [Upload Image](#upload-image)
	- [Transform Image](#transform-image)
	- [Search images](#search-images)
	
- [Statistics](#statistics)
	- [Get Statistics](#get-statistics)
	
- [User](#user)
	- [Create user](#create-user)
	


# Authentication

All endpoints requires authentication.
Live Transform (GET /users/:user/images/:imageId.jpg) and Get File Contents (GET /users/:user/files/:path) has its own type of authentication,
the requests needs to be signed with an apiKey.

## Example Signed request

Signing can be done with either the master apikey or the transform apiKey.
Transform apiKey can only be used for signing live transform request.

### PHP

	<?php
	$secret = 'foobar';
	$url = 'http://localhost:3000/users/martinj/images/50.jpg';
	$transformations = array(
		't' => array(
			'strip' => true,
			'rotate' => array('angle' => 40, 'x' => 2, 'y' => 3)
		)
	);
	$url .= '?' . urldecode(http_build_query($transformations));
	$accessToken = hash_hmac('sha256', $url, $secret);
	$url .= '&accessToken=' . $accessToken;

	//the signed url
	echo $url;

### Node.js

	var crypto = require('crypto'),
		qs = require('qs'),
		secret = 'foobar',
		url = 'http://localhost:3000/users/martinj/images/50.jpg',
		transformations = {
			t: {
				strip: true,
				rotate: { angle: 40, x: 2, y: 3}
			}
		};

	url += '?' + decodeURIComponent(qs.stringify(transformations));
	var accessToken = crypto.createHmac('sha256', secret).update(url).digest('hex');

	url += '&accessToken=' + accessToken;

	//the signed url
	console.log(url);


## Authenticate with header

	Authorization: bearer <token>

## Authenticate with parameter

	http://localhost/file/1?accessToken=<token>

## File Uploads

All files are uploaded in multipart/form-data POST requests.

# File

## Delete File



	DELETE /files/:id


### Parameters

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| id			| Number			|  <p>File ID</p>							|

### Examples

CURL example:

```
CURL example:
curl -i -X DELETE -H 'Authorization: bearer foobar' http://localhost:3000/files/96

```

### Success Response

Success-Response (example):

```
Success-Response (example):
HTTP/1.1 204 No Content

```
## Get File Details



	GET /files/:key


### Parameters

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| key			| Number,String			|  <p>File ID or Path</p>							|

### Examples

CURL example:

```
CURL example:
curl -i -H 'Authorization: bearer foobar' http://localhost:3000/files/96

```

### Success Response

Success-Response (example):

```
Success-Response (example):
HTTP/1.1 200 OK
{
   "id": 96,
   "user_id": 1,
   "path": "/css",
   "md5": "694529d86992b8995800d8935779cd00",
   "content_type": "text/css",
   "original_url": "https://locahost:3000/files/css/style.css",
   "created": "2013-08-21T09:30:50.068Z",
   "updated": "2013-08-21T09:30:50.073Z"
}

```
## Get File Content

<p>Get file content</p><p>Note! This function doesn&#39;t use the regular authentication, each unique url must be signed if files are private.See examples under &quot;Example Signed request&quot; in the General section.</p>

	GET /users/:user/files/:path


### Parameters

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| user			| String			|  <p>the username of the file owner</p>							|
| path			| String			|  <p>the path user for the file</p>							|
| accessToken			| String			|  <p>signed token for the url, see examples in &quot;Example Signed request&quot; in the General section.</p>							|

### Examples

CURL example:

```
CURL example:
curl -i http://localhost:3000/users/test/files/css/style.css

```

### Success Response

Success-Response

```
Success-Response
The file content

```
## Upload / Modify File

<p>The file should be supplied as multipart/form-data (HTTP form file upload) in the request.Multipart/form-data uploads will use the Content-Type in the body.Upload file from remote URL requires this POST to have Content-Type: application/json and its parameters in the body.</p>

	POST /files/:path

### Headers

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| x-ass-acl			| String			| **optional** <p>ACL value (public, private), defaults to private. If not set when overwriting existing file it will always set private.</p>							|
| Cache-Control			| String			| **optional** <p>Specify caching behavior for this file.</p>							|

### Parameters

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| path			| String			|  <p>The path on the storage server to put this including filename.</p>							|
| url			| String			| **optional** <p>url if set it will download the image from this url.</p>							|
| contentType			| String			| **optional** <p>Set content type to use with this file. Specifying content-type will always take precedence.</p>							|

### Examples

Upload File:

```
Upload File:
curl -i -X POST -H 'Authorization: bearer foobar' -F "file=@style.css" \
http://localhost:3000/files/css/style.css

```
Upload from URL and make it public:

```
Upload from URL and make it public:
curl -i -X POST -H 'Authorization: bearer foobar' -H 'x-ass-acl: public' \
-d '{ "url": "http://foo.com/style.css", "contentType": "text/css" }' \
http://localhost:3000/files/css/style.css

```
Modify ACL on existing file:

```
Modify ACL on existing file:
curl -i -X POST -H 'Authorization: bearer foobar' -H 'x-ass-acl: private' \
http://localhost:3000/files/css/style.css

```

### Success Response

Success-Response (example):

```
Success-Response (example):
HTTP/1.1 201 Created
{
   "user_id": 1,
   "path": "css/style.css",
   "original_url": "https://locahost:3000/files/css/style.css",
   "content_type": "text/css",
   "public": true,
   "id": 96,
   "created": "2013-08-21T09:30:50.068Z",
   "updated": "2013-08-21T09:30:50.073Z"
}

```
## Upload File

<p>Upload file. File contents in body, Content-Type in the header will be used.</p>

	PUT /files/:path

### Headers

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| x-ass-acl			| String			| **optional** <p>ACL value (public, private), defaults to private. If not set when overwriting existing file it will always set private.</p>							|
| Cache-Control			| String			| **optional** <p>Specify caching behavior for this file.</p>							|

### Parameters

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| path			| String			|  <p>The path on the storage server to put this including filename.</p>							|

### Examples

Upload File:

```
Upload File:
curl -i -X PUT http://127.0.0.1:3000/files/test/testo.jpg \
--data-binary @testo.jpg -H "Content-Type: image/jpg" -H "Authorization: bearer foobar"

```

### Success Response

Success-Response (example):

```
Success-Response (example):
HTTP/1.1 201 Created
{
   "user_id": 1,
   "path": "css/style.css",
   "original_url": "https://locahost:3000/files/test/test.jpg",
   "content_type": "image/jpg",
   "public": true,
   "id": 96,
   "created": "2013-08-21T09:30:50.068Z",
   "updated": "2013-08-21T09:30:50.073Z"
}

```
## Search Files



	GET /files


### Parameters

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| column			| String			| **optional** <p>The parameter name should be the name on the column you wish to search for. For each column added and value separated with , it uses AND in the search between columns. Value can start with &lt;,&gt;,&lt;=,&gt;= to determine how to match. If value contains % it will match partially. Value can also be !null. Column values can be separated with | to use OR.</p>							|
| fields			| String			| **optional** <p>CSV list of fields to return</p>							|
| orderBy			| String			| **optional** <p>Order by field</p>							|
| order			| String			| **optional** <p>What order to use in conjunction with orderBy parameter. Valid values asc,desc</p>							|
| limit			| String			| **optional** <p>Limit the result. Either a number for how many records to show or csv for setting offset. Same as SQL</p>							|

### Examples

CURL example:

```
CURL example:
curl -i -H 'Authorization: bearer foobar' http://localhost:3000/files?path=/css

```

### Success Response

Success-Response (example):

```
Success-Response (example):
HTTP/1.1 200 OK
[{
   "id": 96,
   "user_id": 1,
   "path": "/css",
   "md5": "694529d86992b8995800d8935779cd00",
   "content_type": "text/css",
   "original_url": "https://locahost:3000/files/css/style.css",
   "created": "2013-08-21T09:30:50.068Z",
   "updated": "2013-08-21T09:30:50.073Z"
}]

```
# Image

## Delete File



	DELETE /images/:id


### Parameters

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| id			| Number			|  <p>Image ID</p>							|

### Examples

CURL example:

```
CURL example:
curl -i -X DELETE -H 'Authorization: bearer foobar' http://localhost:3000/images/96

```

### Success Response

Success-Response (example):

```
Success-Response (example):
HTTP/1.1 204 No Content

```
## Get Image Details



	GET /images/:id


### Parameters

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| id			| Integer			|  <p>Image Id</p>							|

### Examples

CURL example:

```
CURL example:
curl -i -H 'Authorization: bearer foobar' http://localhost:3000/images/1

```

### Success Response

Success-Response (example):

```
Success-Response (example):
HTTP/1.1 200 OK
{
   "id": 1,
   "user_id": 1,
   "md5": "694529d86992b8995800d8935779cd00",
   "original_url": "http://localhost:3000/martinj/images/2013/07/11/1.jpg",
   "width": 2560,
   "height": 1600,
   "name": "foobar.jpg",
   "title": null,
   "description": null,
   "author": null,
   "created": "2013-07-11T13:00:05.000Z",
   "updated": "2013-07-11T13:00:05.000Z"
}

```
## Live Transform

<p>Url based image manipulations that serves the generated image.All the transformation parameters should be inside t parameter. e.gt[strip]=true&amp;t[rotate][angle]=40&amp;t[rotate][x]=10&amp;t[rotate][y]=10&amp;t[resize][width]=40</p><p>The transformation actions are applied in the same order as they are received.</p><p>Note! This function doesn&#39;t use the regular authentication, each unique url must be signed.See examples under &quot;Example Signed request&quot; in the General section.</p>

	GET /users/:user/images/:imageId.jpg


### Parameters

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| user			| String			|  <p>The username owning the image</p>							|
| imageId			| Number			|  <p>The image you want to make transformations on.</p>							|
| accessToken			| String			|  <p>signed token for the url, see examples in &quot;Example Signed request&quot; in the General section.</p>							|
| t			| Object			| **optional** <p>This is the container for all the transformation parameters. e.g [strip]=true&amp;t[rotate][angle]=40</p>							|
| rotate			| Object			| **optional** <p>Rotate image, properties { angle: 0, x: 0, y: 0 }</p>							|
| crop			| Object			| **optional** <p>Properties { width: 2560, height: 1013, x: 0, y: 0 }</p>							|
| resize			| Object			| **optional** <p>Properties { width: 470, height: 186, flag: &#39;&gt;&#39; }, available flags &lt;,&gt;,!,^ for more info about flags check <a href="http://www.imagemagick.org/Usage/resize/#noaspect">http://www.imagemagick.org/Usage/resize/#noaspect</a></p>							|
| sharpen			| Object			| **optional** <p>Properties { mode: &#39;variable&#39; }, available modes are: light, moderate, strong, extreme, off.</p>							|
| strip			| Boolean			| **optional** <p>Strip image of all profiles and comments.</p>							|
| quality			| Number			| **optional** <p>Compression quality</p>							|
| extent			| Object			| **optional** <p>Properties { width: 2560, height: 1013 }</p>							|
| gravity			| String			| **optional** <p>Valid values <a href="http://www.imagemagick.org/script/command-line-options.php#gravity">http://www.imagemagick.org/script/command-line-options.php#gravity</a></p>							|
| force			| Boolean			| **optional** <p>By default it will only generate image if it doesn&#39;t exist, specifying true here will force new image transformation.</p>							|

### Examples

CURL example:

```
CURL example:
Urldecoded parameters: t[strip]=true&t[rotate][angle]=40&t[rotate][x]=10&t[rotate][y]=10&t[resize][width]=40
curl -i -H 'Authorization: bearer foobar' \
"http://localhost:3000/users/martin/images/20.jpg?t%5Bstrip%5D=1&t%5Brotate%5D%5Bangle%5D=40&t%5Brotate%5D%5Bx%5D=10&t%5Brotate%5D%5By%5D=10&t%5Bresize%5D%5Bwidth%5D=40&accessToken=336d659ca768f0549b4706a41f7768f0b1e83241c6890a4b52776545203615c9"

```

### Success Response

Success-Response

```
Success-Response
The transformed image data

```
## Upload Image

<p>The file should be supplied as multipart/form-data (HTTP form file upload) in the request.Only JPG and PNG images are supported.</p>

	POST /images


### Parameters

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| url			| String			| **optional** <p>url if set it will download the image from this url.</p>							|

### Examples

CURL example:

```
CURL example:
curl -i -X POST -H 'Authorization: bearer foobar' -F "file=@choco.jpg" http://localhost:3000/images

```

### Success Response

Success-Response (example):

```
Success-Response (example):
HTTP/1.1 201 Created
{
  "name": "choco.jpg",
  "user_id": 1,
  "created": "2013-08-20T13:36:50.638Z",
  "md5": "a771b0ecffa0c08c8e488d8ed7aeec50",
  "width": 400,
  "height": 300,
  "title": null,
  "description": null,
  "author": null,
  "id": 16,
  "original_url": "https://locahost:3000/foobar/images/2013/08/20/16.jpg"
}

```
## Transform Image

<p>Make image manipulations on existing image.The transformation actions are applied in the same order as they are received.</p>

	POST /images/:id/transform


### Parameters

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| id			| Number			|  <p>The image you want to make transformations on.</p>							|
| rotate			| Object			| **optional** <p>Rotate image, properties { angle: 0, x: 0, y: 0 }</p>							|
| crop			| Object			| **optional** <p>Properties { width: 2560, height: 1013, x: 0, y: 0 }</p>							|
| resize			| Object			| **optional** <p>Properties { width: 470, height: 186, flag: &#39;&gt;&#39; }, available flags &lt;,&gt;,!,^ for more info about flags check <a href="http://www.imagemagick.org/Usage/resize/#noaspect">http://www.imagemagick.org/Usage/resize/#noaspect</a></p>							|
| sharpen			| Object			| **optional** <p>Properties { mode: &#39;variable&#39; }, available modes are: light, moderate, strong, extreme, off.</p>							|
| strip			| Boolean			| **optional** <p>Strip image of all profiles and comments.</p>							|
| quality			| Number			| **optional** <p>Compression quality</p>							|
| extent			| Object			| **optional** <p>Properties { width: 2560, height: 1013 }</p>							|
| gravity			| String			| **optional** <p>Valid values <a href="http://www.imagemagick.org/script/command-line-options.php#gravity">http://www.imagemagick.org/script/command-line-options.php#gravity</a></p>							|
| force			| Boolean			| **optional** <p>By default it will only generate image if it doesn&#39;t exist, specifying true here will force new image transformation.</p>							|

### Examples

CURL example:

```
CURL example:
curl -i -X POST http://localhost:3000/images/16/transform \
-H 'Authorization: bearer foobar' \
-H 'Content-Type: application/json' \
-d '{ "resize": { "width": 10, "height": 10 }, "strip": true }'

```

### Success Response

Success-Response (example):

```
Success-Response (example):
HTTP/1.1 201 Created
{
  "image_id": 16,
  "hash": "0fea10d98df15639d663023a0777ccb0",
  "actions": {
    "resize": {
      "width": 10,
      "height": 10
    },
    "strip": true
  },
  "width": 10,
  "height": 5
}

```
## Search images



	GET /images


### Parameters

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| column			| String			| **optional** <p>The parameter name should be the name on the column you wish to search for. For each column added and value separated with , it uses AND in the search between columns. Value can start with &lt;,&gt;,&lt;=,&gt;= to determine how to match. If value contains % it will match partially. Value can also be !null. Column values can be separated with | to use OR.</p>							|
| fields			| String			| **optional** <p>CSV list of fields to return</p>							|
| orderBy			| String			| **optional** <p>Order by field</p>							|
| order			| String			| **optional** <p>What order to use in conjunction with orderBy parameter. Valid values asc,desc</p>							|
| limit			| String			| **optional** <p>Limit the result. Either a number for how many records to show or csv for setting offset. Same as SQL</p>							|

### Examples

CURL example:

```
CURL example:
curl -i -H 'Authorization: bearer foobar' "http://localhost:3000/images?name=choco.jpg"

```

### Success Response

Success-Response (example):

```
Success-Response (example):
HTTP/1.1 200 OK
[{
  "name": "choco.jpg",
  "user_id": 1,
  "created": "2013-08-20T13:36:50.638Z",
  "md5": "a771b0ecffa0c08c8e488d8ed7aeec50",
  "width": 400,
  "height": 300,
  "title": null,
  "description": null,
  "author": null,
  "id": 16,
  "original_url": "https://locahost:3000/foobar/images/2013/08/20/16.jpg"
}]

```
