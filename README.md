# TikTok API Returns Empty Response due to lack of harmless headers .
Hi,

In these repos you can see people complaining about this behaviour:

https://github.com/davidteather/TikTok-Api/issues/1090

https://github.com/davidteather/TikTok-Api/issues/974

But why does it return empty response without these harmless parameters: `msToken, signature and X-bogus` parameters.

The main reason for this is because of using `XHR()` API and communicating with REST API with the server. 

Let's consider this pseudo code I have written:
```javascript
// Create a new XHR object
var xhr = new XMLHttpRequest();

// Configure the request
xhr.open('POST', 'https://live-backstage.tiktok.com/api/endpoint', true); // Example TikTok API endpoint
xhr.setRequestHeader('Content-Type', 'application/json'); // Set the CT header

// Construct the request body with required parameters
var requestBody = {
  msToken: 'your_msToken_value',
  signature: 'your_signature_value',
  'X-bogus': 'your_X-bogus_value'
};

// Set up a callback function to handle the response
xhr.onreadystatechange = function () {
  if (xhr.readyState === XMLHttpRequest.DONE) {
    if (xhr.status === 200) {
      // Request was successful, handle the response data
      var responseData = JSON.parse(xhr.responseText);
      console.log(responseData);
    } else {
      // Handle errors or other status codes
      console.error('Error:', xhr.status);
    }
  }
};

// Send the request with the JSON-encoded request body
xhr.send(JSON.stringify(requestBody));
```
The pseudocode I provided is an example of how you will structure a client-side JavaScript code using XHR API to send a POST request to such API endpoint.

`requestBody` object provides three properties. However missing value of any harmless `POST` or `GET` based parameter is going to return 200 OK empty response just like `HEAD` based request does.

However this happens with even harmless HTTP headers. So, here's my conclusion of why this actually happens, The server is actually performing strict parameters validation (they probably are doing this against such CSRF attacks which actually makes no sense as these parameters are global-wide on TikTok and are absolutely harmless, they don't come from cookies or anything, it's being generated but fun fact is they have no expiry time / long expiration time and it is being sent in URL which means it can be exposed.) , and if a required parameter like msToken is missing, it responds with a success status (200 OK) but does not provide a response body to indicate that the request was incomplete or incorrect.

Here's the little Node.js pseudo code of how it happens at the back-end when API starts the communication with the server:
```javascript
app.use(bodyParser.json());

app.post('/api/endpoint', (req, res) => {
  const requestBody = req.body;

  // Check if 'msToken' parameter is missing
  // Further more, this applies only for this parameter, but it's same for other parameters as well
  if (!requestBody.msToken) {
    // Respond with a 200 OK status and no response body
    return res.status(200).send();
  } else {
    // Process the request normally and return a response
    // For demonstration purposes, we'll just echo back the request data
    return res.json(requestBody);
  }
});
```
**So, overall if you come with this issue and you have a valid vulnerability, you should report the issue but it will be considered as `AC:H` because of this**:

> (they probably are doing this against such CSRF attacks which actually makes no sense as these parameters are global-wide on TikTok and are absolutely harmless, they don't come from cookies or anything, it's being generated but fun fact is they have no expiry time / long expiration time and it is being sent in URL which means it can be exposed.)
