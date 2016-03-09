# zn-router

Routing for Zengine backend services (unofficial and experimental).

# Install

```sh
$ npm install zn-router
```

# Use

You could create a file where you register your routes, e.g. `my-service-router.js`

```js
var znRouter = require('zn-router');

// register routes... (explained below)

module.exports = znRouter.dispatch;
```

And then add this to your `plugin.js`

```js
var dispatch = require('./src/my-service-router.js');

exports.run = function(eventData) {
	dispatch(eventData);
};
```

## GET Requests

```js

// GET: /workspaces/123/pluginNamespace/serviceName

znRouter.get('/', function(request, response) {

  var workspaceId = request.params.workspaceId;

  // getSomething should return a promise
  // If promise is resolved,
  // service will respond with status 200 and promise response as json response
  // If promise is rejected,
  // service will respond with status 500 (unless you specify otherwise – see below)
  return getSomething(workspaceId);

});
```

## POST Requests

```js
// POST: /workspaces/123/pluginNamespace/serviceName/webhooks

znRouter.post('/webhooks', function(request, response) {

	var body = request.body;
	var data = body.data[0];

	var activityId = data.id;
	var workspaceId = data.workspace.id;
	var webhookSecret = request.headers['x-zengine-webhook-key'];

	var info = {
		activityId: activityId,
		workspaceId: workspaceId,
		webhookSecret: webhookSecret
	};

	return processWebhook(info);
});

```

## Error handling

For both `get` and `post` registration, you can catch errors before they are handled by znRouter.

```js
znRouter.get('/apples', function(request, response) {

  var onError = function(err) {
  
    // You can catch specific errors and return a different status code and response
    if (err.message === 'SomeKnownError') {
      // don't forget to return
      return response.status(400).send({
        message: 'Bad Request'
      });
    }
    
    // Unknown error: Re-throw, so znRouter handles that for you
    throw err;
  };

  return getApples().catch(onError);

});
```
