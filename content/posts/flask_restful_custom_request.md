---
title: Msgpack in Flask-RESTful
date: 2017-01-08
author: Kendrick Tan
categories: ["python"]
---

### **[Link to the boilerplate project](https://github.com/kendricktan/flaskrestful-custom-request)**

Recently I've been building a RESTful microservice, one the the requirements that the microservice has is that it has [MessagePack](https://msgpack.org/index.html) support (it's like JSON but smaller). [Flask](https://flask.pocoo.org) doesn't support MessagePack by default, and the documentation on extending [Flask](https://flask.pocoo.org) for custom `Request` types was lacking at the time of this writing, and as such I've decided to write a post to serve as a guideline to whomever who's running into the same issue.

I'll be using [Flask](https://flask.pocoo.org) and [Flask-RESTful](https://flask-restful-cn.readthedocs.io/en/0.3.4/) in this tutorial.

### **Extending Flask's Request Class**

Unfortunately since `Flask-RESTful` inherits its `Request` parsing from `Flask`, we can't just extend on `Flask-RESTful` and expect it to be able to parse our custom `Request` type. As such, we'll need to subclass `Flask` to specify our custom `Request` class (which is also a subclass of `Flask`'s `Request` class).

```python
from flask import Flask
from flask import Request, _request_ctx_stack

class RequestWithMsgPack(Request):
    """
    Extending on Flask's Request class to support msgpack mimetype
    """
    pass

class FlaskWithMsgPackRequest(Flask):
    """
    Extending on Flask to specify the usage of our custom Request class
    """
    request_class = RequestWithMsgPack
```

### **Adding MessagePack Parsing Functionality**

Next thing we need to do is write a custom `MessagePack` parser in our custom `RequestWithMsgPack` class.

*Important*: Note down the function used to parse the custom `Request` class you have in mind, in our case its the `def msgpack(...)`  function. This will be discussed later on.

```python    
import msgpack
from flask import Flask
from flask import Request, _request_ctx_stack
from flask.wrappers import _get_data
from werkzeug.exceptions import BadRequest


class RequestWithMsgPack(Request):
    """
    Extending on Flask's Request class to support msgpack mimetype
    """

    @property
    def is_msgpack(self):
        """
        Checks if request is msgpack type or not.
        """
        mt = self.mimetype
        return mt.startswith('application/') and mt.endswith('msgpack')

    def msgpack(self, force=False, silent=False):
        """
        NOTE: This function name needs to be the same name specified on the
        'location' variable of the request parser. e.g.
        parser.add_argument('data', location='msgpack') `location needs to have the same
                                                        name as the callable function
        Parses the incoming request data and decodes it from msgpack to python
        __dict__ type. By default this function will return `None` if the mimetype
        is not `application/msgpack` but can be overridden by the ``force`` parameter.
        If parsing fails the
        :param force: if set to ``True`` the mimetype is ignored
        :param silent: if set to ``True`` this method will fail silently and return ``None``
        """
        if not (force or self.is_msgpack):
            return None

        # Convert to utf-8 by default
        request_charset = self.mimetype_params.get('charset', 'utf-8')

        try:
            data = _get_data(self, False)
            rv = msgpack.unpackb(data, encoding=request_charset)

        except ValueError as e:
            if silent:
                return None
            else:
                rv = self.on_msgpack_loading_failed(e)

        return rv

    def on_msgpack_loading_failed(self, e):
        """
        Called if decoding of msgpack data failed
        """
        ctx = _request_ctx_stack.top

        if ctx is not None and ctx.app.config.get('DEBUG', False):
            raise BadRequest('Failed to decode msgpack object: {0}'.format(e))

        raise BadRequest()

class FlaskWithMsgPackRequest(Flask):
    """
    Extending on Flask to specify the usage of our custom Request class
    """
    request_class = RequestWithMsgPack
```

### **Using MessagePack Parsing in Flask-RESTful Routes** 

Remember the notice from before to note down the function used to parse the custom `Request` class? It'll now be demonstrated on `Flask-RESTful`'s `Router`.

To invoke our custom `Request` class in `Flask-RESTful`, we need specify the name of our function defined above as the `location` parameter in the `RequestParser` (**line 14**). For example, if we renamed our `msgpack` function above to `custom_msgpack`, then instead of `location='msgpack'`, it'll be `location='custom_msgpack` on **line 14**.

```python
class HelloMsgPack(Resource):
    """
    Class containing our endpoints
    """

    def get(self):
        return {'hello': 'You sent a GET request'}

    def post(self):
        parse = reqparse.RequestParser()
        # Important: define your 'location' to whatever function you
        # defined in your custom Request class
        parse.add_argument('data', location='msgpack', help='data in msgpack form', required=True)

        args = parse.parse_args()

        return {'Received data': args['data']}
```

### **Putting them together**

Since we've just subclasses `Flask` and `Flask-RESTful`, the way they're used is still the same, just with added functionality.

```python
from flask_restful import Api

app = FlaskWithMsgPackRequest(__name__)
api = Api(app)

api.add_resource(HelloMsgPack, '/')

if __name__ == "__main__":
    app.run(debug=False)
```

### **Final Thoughts**

I've tried to keep this tutorial as simple as possible and omitted the `Response` of `MessagePack` type objects as it is well documented at the time of writing. The [boilerplate project](https://github.com/kendricktan/flaskrestful-custom-request) does contain examples on how to write custom `Response` classes if a reference is needed.