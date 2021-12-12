# Quick start

Pixble transforms images into professional edited images. Our API utilizes state of the art Artificial Intelligence to enhance lighting, color, tone and sharpen blurry images.


## API Key
Sign up an account at [here](https://pixble.com/signup). Navigate to [/key](https://pixble.com/keys) and generate a new API key.


## Authentication
Pixble use an API key to authenticate requests. Authentication to the API is performed via parameters. Provide your API key when you send request as `key=[API key]`. You do not need to provide a password.


## Making a request
To process images, make a `POST` request to the endpoint with the parameter `key` being set to your API key along with other operations. 

Depending on image size, the processing time varies. You can use a Webhook to make sure it is handled when your image is processed.

**API Endpoint**
```
https://api.pixble.com/job/new
```

**Request body**

**Bash**
```bash
curl -X POST https://api.pixble.com/job/new \
	-H "Content-Type: application/json" \
    -d "{
    		'task': 9 \
    		'key': [Your API Key] \
   			'data': [Base64 image data] \
   		}"

```

**Javscript**
```Javascript
const fs = require ("fs")
const request = require('request');

const data = fs.readFileSync('./image.jpeg', {encoding: 'base64'});

request ("https://api.pixble.com/job/new", {
	method: "POST",
	headers: {"Content-Type': 'application/json"} 
	body: JSON.stringify({
		"task": 9,
		"key": [Your API Key],
		"data": data
	})
}).then((res) => {

	console.log (res)
})

```

**Python**
```Python
import requests
import base64

image = open('./image.jpeg', 'rb')

data = base64.encodestring(image.read())

body = {
	"task": 9,
	"key": [Your API Key],
	"data": data
}

res = requests.post("https://api.pixble.com/job/new", data = body)

print (res)
```

## Tasks
The request needs to specify which task to perform on the image.

|Task|Name|Description|
|--|--|--|
|9|Pixble Magic|Automatically fix lighting, color, tone. Sharpen blurry images|

## Parameters

** Task 9: Pixble Magic**
- `clarify_image` Boolean: 1 and 0 (default: 1). Toggle AI pixel enhancement option. It automatically clarifies blurry images by using state of the art AI algorithm. 
- `adv_illumin` Float: 0 to 5 (default: 1.3). Adjust the illumination of the image with AI algorithm. 0 is no adjustment and 5 is being the maximum brightness. 
- `color_correction` Boolean: 1 and 0 (default: 1). Advanced color and tone correction option. It automatically balance all the colors and tone to make sure they look stunning.

## Response
Once the request is made, a response is sent as following:
```
{
  error_code: '',
  error_message: '',
  status: true,
  result: {
    sid: 'si_3CRjTFLXIpZTYxraY2lP1',
    job_id: 'jb_M0M2NfgnXIjL1VnyQVs8v',
    credit_left: 137.8,
    credit_used: 1.2,
    uuid: '39b394cf-649c-4ccb-a0b4-f83e2103737e'
  },
  timestamp: 1639335021377
}
```

The parameters below are important for the request.
- `status` Boolean. It is to describe if the request is successful
- `job_id` Varchar. It is the unique job id to retrieve the result image in the later time


## Retrieve the result
Depending on the size of the image, the processing can vary from 30 seconds to 15 minutes. To check the status of the job, make a GET request to the following endpoint with your job id and your API key.

**API Endpoint**
```
https://api.pixble.com/job/info/[job id]?key=[API key]
```

The response is as followed. For the both input and output file, we store them for 60 days once they are done. 
- `status` int. There are different status code for the request. Please see below section. `11` is sccuessful. `312` is processing. `912` is failed.
```
{
   "error_code":"",
   "error_message":"",
   "status":true,
   "result":{
      "job_id":"jb_M0M2NfgnXIjL1VnyQVs8v",
      "task_id":"9",
      "status":11,
      "created_at":"2021-12-12T18:50:21.128Z",
      "last_update":"2021-12-12T18:51:31.931Z",
      "completed_at":"2021-12-12T18:51:31.931Z",
      "created_via":"key",
      "input_file":"fi_T-Yq6xNxIl4FZZCIfu4BK",
      "output_file":"fi_9WhrmvOrkqk-tbqZlXT2V",
      "files":[
         {
            "file_id":"fi_T-Yq6xNxIl4FZZCIfu4BK",
            "job_id":"jb_M0M2NfgnXIjL1VnyQVs8v",
            "source":"USER_API_UPLOAD",
            "url":null,
            "content_type":"image/jpeg",
            "size":12872,
            "dim":"100x125",
            "link":"https://api.pixble.com/media/get/fi_T-Yq6xNxIl4FZZCIfu4BK",
            "preview":"https://api.pixble.com/media/preview/fi_T-Yq6xNxIl4FZZCIfu4BK"
         },
         {
            "file_id":"fi_9WhrmvOrkqk-tbqZlXT2V",
            "job_id":"jb_M0M2NfgnXIjL1VnyQVs8v",
            "source":"OUTPUT",
            "url":null,
            "content_type":"image/jpeg",
            "size":9957,
            "dim":"100x125",
            "link":"https://api.pixble.com/media/get/fi_9WhrmvOrkqk-tbqZlXT2V",
            "preview":"https://api.pixble.com/media/preview/fi_9WhrmvOrkqk-tbqZlXT2V"
         }
      ]
   },
   "timestamp":1639335895382
}
```
To download the images, make a GET request to the following endpoint with your file id or job id and API key 

```
https://api.pixble.com/media/get/fi_9WhrmvOrkqk-tbqZlXT2V?key=[API key]
```

### Webhooks
The easiest way to be notified when the job is done is to setup a callback. You can setup a email callback and a webhook call at the [account](https://pixble.com/account) page after you login. 

The webhook send the POST request once the job is done. If your server responds with error (status code ≥ 400), the next retry will occur in 10 seconds. 1 minutes after the previous request and 2 minutes afterward.


