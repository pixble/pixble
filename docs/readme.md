## Quick start

Pixble transforms images into professional edited images. Our API utilizes state of the art Artificial Intelligence to enhance lighting, color, tone and sharpen blurry images.


### API Key
Sign up an account at [here]("https://pixble.com/signup"). Navigate to [/key]([here]("https://pixble.com/keys")) and generate a new API key.


### Authentication
Pixble use an API key to authenticate requests. Authentication to the API is performed via parameters. Provide your API key when you send request as `key=[API key]`. You do not need to provide a password.


### Making a request
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