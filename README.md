# Python REST API
## Setup
`pip3 install -r requirements.txt`

using **requirements.txt**, install the appropriate python modules such as flask, flask restful, sql alchemy...

## Testing
We can use the browser, but it's also simple to test from the terminal:
```
import requests

home = "http://127.0.0.1:5000\"

response = requests.get(home + "helloworld")
print(response.json)
```


## Begin coding
```
from flask import Flask
from flask_restful import Api,Resource
```

next we need to 'create an app'.  this is a flask application is it not?
```
app = Flask(__name__)
api = Api(app)
```

the Api function wraps our flask app object in the restful api 

```
if __name__ = "__main__":
	app.run(debug=True) # don't use in production!
```

next we're going to establish some **resources**

```
class HelloWorld(Resource): # this class inherits from Resource.  there will be a few methods that it can then override
	def get(self):
		return {"Hello World"}

	def put(self):
		return {"yo"}
```

then we can add this resource to our restful api object

```
api.add_resource(HelloWorld, "/helloworld") the first arg is the resource, the second is the route
```

## Defining parameters
we use angle brakckets in the route string and write a type wrapped in angle brackets:
`api.add_resource(HelloWorld, "/helloworld/<string: name>)"`

we can then refer to this parameters in the appropriate method, eg the get method

```
def get(self, name):
	return {"data": "hello" + name}
```

this can then be tested by making a request to "/helloworld/Ross"

## How to create new objects
first of all we need to be able to retrieve data from the body of the HTTP request

```
from flask import request
...
request.form['fieldname']
```

these values can be passed in the body in the test script using:

`respose = requests.put(BASE + "whatever/" {"fieldname": "value"})`

there is an easier method however, using **reqparse**

`band_put_args = reqparse.RequestParser() # instantiate a request parser object`

this object can automatically parse through the requests coming in and make sure they fit what we're expecting in our Resource

`bands_put_args.add_argument("fieldname", type=str, help="this is error test")`

now in our **put** method in our Resource, we parse these arguments:

`args = band_put_args.parse_args()`

this will parse the data that we pass in using the request thusly:

`response = requests.put(home + "bands/1", {"name":"xyz", "style":"abc"}`

the arguments are in the **body** of the request, while the ID number that we are currently giving it is in the **route**

We can add `required = True` to the `add_argument` function calls when we're 'putting args'  the request parser object.  This will give us some better error reporting if any required values are missing from the body of the request

***Note - we can add a secondary return value in our Resource class' methods, which is the HTTP response code, eg 200.  for PUT we will use 201 to indicate a resource was created***

## abort
`from flask_restful import abort`

we use `abort()` to send a custom error message (the second parameter of the abort function).  It's neat to create functions to handle tests and aborts.

the first parameter is the status code.

`abort(404, message="error text")`

***Note - when dealing with requests and responses for testing, don't go looking for response.json() if you're just returning a status code etc***

## SQLAlchemy
a lightweight database

```
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///dbname.db'
db = SQLAlchemy(app)
```
first we must alter our flask app's configuration to add a URI that points to our database.  the config index must be in all caps here

`sqlite:///dbname.db` points to the location of the physical db file.  this is a relative path within your application, so if you create another folder inside the working directory you refer to it thus:
`sqlite:///folder/dbname.db`

to build the database:
`db.create_all()`
*we only do this ONCE*.  otherwise if we re-initialise it, we will overwrite our data.

Before we create_all the database, we want to build our database models using classes:

```
class BandModel(db.Model): # db.Model is what this class is inheriting from
	id = db.Column(db.Integer, primary_key=True)
	name = db.Column(db.String(50), nullable=False)
	style = db.Column(db.String(50))
	status = db.Column(db.String(50))

	def __repr__(self): # what'll be returned if we ask for the representation of the object
		return f"Band(name={name} is {status})"
```

## Resource Fields
```
from flask_restful import ields, marshal_with
resource_fields = {
	'id': fields.Integer,
	'name': fields.String,
	'style': fields.String,
	'status': fields.status
}
```
this dictionary "resource_fields" defines the fields from the model that we want to return, if we return that object.

then when interacting with one of these objects - which is returned when we query the sqlalchemy db - we use this tag over our Resource method:
```
@marshal_with(resource_fields)
def get(self, band_id):
	result = BandModel.query.filter_by(id=band_id).first # get the first of the results filtered by the given parameter
	return result
```

the @ tag says "when we return, serialise the returned object using the fields in the dictionary 'resource_fields'".  this will make it look like JSON in effect.

we can then access this using our arg parser and use the dictionary-like values in a constructor - to make an object:

```
args = band_put_args.parse_args()
band = BandModel(id=band_id, name=args['name'], style=args['style'], status=args['status'])

db.session.add(band)
db.session.commit()
```

Above we have created a new band using the parsed response, and then added it to the table that matches the Band Model.  we then commit the changes so they're stored.

## Update
rather than being called 'update', the function is now called **patch**

```
def patch(self, band_id):
```

inside this function we want to find the appropriate record and then update:
```
def patch(self, band_id):
	result = BandModel.query.filter_by(id=band_id).first()
	if not result:
		abort(404, message="band not found")
	
	args = band_update_args.parse_args()
	
	if args['name']:
		result.name = args['name']
	if args['style']:
		result.style = args['style']
	if args['style']:
		result.status = args['status']
	
	db.session.commit()
	
	return result, 204
```

and to test this we simply adjust the request body of our HTTP request:

`response = requests.patch(BASE + "bands/2", {"status": "alive"})`

which will change the status of band 2, but no other values

#python #REST #API #flask #flask-restful #sqlalchemy
