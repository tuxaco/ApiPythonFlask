### Flask[](https://realpython.com/api-integration-in-python/#flask "Permanent link")

[Flask](https://realpython.com/introduction-to-flask-part-1-setting-up-a-static-site/)  is a Python microframework used to build web applications and REST APIs. Flask provides a solid backbone for your applications while leaving many design choices up to you. Flask’s main job is to handle HTTP requests and route them to the appropriate function in the application.

**Note:**  The code in this section uses the new  [Flask 2](https://palletsprojects.com/blog/flask-2-0-released/)  syntax. If you’re running an older version of Flask, then use  [`@app.route("/countries")`](https://flask.palletsprojects.com/en/2.0.x/api/#flask.Flask.route)  instead of  [`@app.get("/countries")`](https://flask.palletsprojects.com/en/2.0.x/api/#flask.Flask.get)  and  [`@app.post("/countries")`](https://flask.palletsprojects.com/en/2.0.x/api/#flask.Flask.post).

To handle  `POST`  requests in older versions of  [Flask](https://flask.palletsprojects.com/en/1.1.x/), you’ll also need to add the  `methods`  parameter to  `@app.route()`:

`@app.route("/countries", methods=["POST"])` 

This route handles  `POST`  requests to  `/countries`  in Flask 1.

Below is an example Flask application for the REST API:

```
#app.py
from flask import Flask, request, jsonify

app = Flask(__name__)

countries = [
    {"id": 1, "name": "Thailand", "capital": "Bangkok", "area": 513120},
    {"id": 2, "name": "Australia", "capital": "Canberra", "area": 7617930},
    {"id": 3, "name": "Egypt", "capital": "Cairo", "area": 1010408},
]

def _find_next_id():
    return max(country["id"] for country in countries) + 1

@app.get("/countries")
def get_countries():
    return jsonify(countries)

@app.post("/countries")
def add_country():
    if request.is_json:
        country = request.get_json()
        country["id"] = _find_next_id()
        countries.append(country)
        return country, 201
    return {"error": "Request must be JSON"}, 415
```
    
This application defines the API endpoint  `/countries`  to manage the list of countries. It handles two different kinds of requests:

1.  **`GET /countries`**  returns the list of  `countries`.
2.  **`POST /countries`**  adds a new  `country`  to the list.

**Note:**  This Flask application includes functions to handle only two types of requests to the API endpoint,  `/countries`. In a full REST API, you’d want to expand this to include functions for all the required operations.

You can try out this application by installing  `flask`  with  `pip`:

`$ python -m pip install flask` 

Once  `flask`  is installed, save the code in a file called  `app.py`. To run this Flask application, you first need to set an environment variable called  `FLASK_APP`  to  `app.py`. This tells Flask which file contains your application.

Run the following command inside the folder that contains  `app.py`:

`$ export FLASK_APP=app.py` 

This sets  `FLASK_APP`  to  `app.py`  in the current shell. Optionally, you can set  `FLASK_ENV`  to  `development`, which puts Flask in  **debug mode**:

`$ export FLASK_ENV=development` 

Besides providing helpful error messages, debug mode will trigger a reload of the application after all code changes. Without debug mode, you’d have to restart the server after every change.

**Note:**  The above commands work on macOS or Linux. If you’re running this on Windows, then you need to set  `FLASK_APP`  and  `FLASK_ENV`  like this in the Command Prompt:

`C:\> set FLASK_APP=app.py
C:\> set FLASK_ENV=development` 

Now  `FLASK_APP`  and  `FLASK_ENV`  are set inside the Windows shell.

With all the environment variables ready, you can now start the Flask development server by calling  `flask run`:

`$ flask run
* Serving Flask app "app.py" (lazy loading)
* Environment: development
* Debug mode: on
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)` 

This starts up a server running the application. Open up your browser and go to  `http://127.0.0.1:5000/countries`, and you’ll see the following response:

```
[
    {
        "area": 513120,
        "capital": "Bangkok",
        "id": 1,
        "name": "Thailand"
    },
    {
        "area": 7617930,
        "capital": "Canberra",
        "id": 2,
        "name": "Australia"
    },
    {
        "area": 1010408,
        "capital": "Cairo",
        "id": 3,
        "name": "Egypt"
    }
]
```

This JSON response contains the three  `countries`  defined at the start of  `app.py`. Take a look at the following code to see how this works:

```
@app.get("/countries")
def get_countries():
    return jsonify(countries)
```

This code uses  `@app.get()`, a Flask route  [decorator](https://realpython.com/primer-on-python-decorators/), to connect  `GET`  requests to a function in the application. When you access  `/countries`, Flask calls the decorated function to handle the HTTP request and  [return](https://realpython.com/python-return-statement/)  a response.

In the code above,  `get_countries()`  takes  `countries`, which is a  [Python list](https://realpython.com/python-lists-tuples/), and converts it to JSON with  `jsonify()`. This JSON is returned in the response.

**Note:**  Most of the time, you can just return a Python dictionary from a Flask function. Flask will automatically convert any Python dictionary to JSON. You can see this in action with the function below:

```
@app.get("/country")
def get_country():
    return countries[1]
```

In this code, you return the second dictionary from  `countries`. Flask will convert this dictionary to JSON. Here’s what you’ll see when you request  `/country`:

```
{
    "area": 7617930,
    "capital": "Canberra",
    "id": 2,
    "name": "Australia"
}` 
```
This is the JSON version of the dictionary you returned from  `get_country()`.

In  `get_countries()`, you need to use  `jsonify()`  because you’re returning a list of dictionaries and not just a single dictionary. Flask doesn’t automatically convert lists to JSON.

Now take a look at  `add_country()`. This function handles  `POST`  requests to  `/countries`  and allows you to add a new country to the list. It uses the Flask  [`request`](https://flask.palletsprojects.com/en/1.1.x/quickstart/#the-request-object)  object to get information about the current HTTP request:

```
@app.post("/countries")
def add_country():
    if request.is_json:
        country = request.get_json()
        country["id"] = _find_next_id()
        countries.append(country)
        return country, 201
    return {"error": "Request must be JSON"}, 415
```

This function performs the following operations:

1.  Using  [`request.is_json`](https://flask.palletsprojects.com/en/1.1.x/api/?highlight=is_json#flask.Request.is_json)  to check that the request is JSON
2.  Creating a new  `country`  instance with  `request.get_json()`
3.  Finding the next  `id`  and setting it on the  `country`
4.  Appending the new  `country`  to  `countries`
5.  Returning the  `country`  in the response along with a  `201 Created`  status code
6.  Returning an error message and  `415 Unsupported Media Type`  status code if the request wasn’t JSON

`add_country()`  also calls  `_find_next_id()`  to determine the  `id`  for the new  `country`:

```
def _find_next_id():
    return max(country["id"] for country in countries) + 1
```

This helper function uses a  [generator expression](https://realpython.com/introduction-to-python-generators/)  to select all the country IDs and then calls  `max()`  on them to get the largest value. It increments this value by  `1`  to get the next ID to use.

You can try out this endpoint in the shell using the command-line tool  [curl](https://curl.se/), which allows you to send HTTP requests from the command line. Here, you’ll add a new  `country`  to the list of  `countries`:

```
$ curl -i http://127.0.0.1:5000/countries \
-X POST \
-H 'Content-Type: application/json' \
-d '{"name":"Germany", "capital": "Berlin", "area": 357022}'

HTTP/1.0 201 CREATED
Content-Type: application/json
...

{
 "area": 357022,
 "capital": "Berlin",
 "id": 4,
 "name": "Germany"
} 
```
This curl command has some options that are helpful to know:

-   **`-X`**  sets the HTTP method for the request.
-   **`-H`**  adds an HTTP header to the request.
-   **`-d`**  defines the request data.

With these options set, curl sends JSON data in a  `POST`  request with the  `Content-Type`  header set to  `application/json`. The REST API returns  `201 CREATED`  along with the JSON for the new  `country`  you added.

**Note:**  In this example,  `add_country()`  doesn’t contain any validation to confirm that the JSON in the request matches the format of  `countries`. Check out  [flask-expects-json](https://pypi.org/project/flask-expects-json/)  if you’d like to validate the format of JSON in Flask.

You can use curl to send a  `GET`  request to  `/countries`  to confirm that the new  `country`  was added. If you don’t use  `-X`  in your curl command, then it sends a  `GET`  request by default:

```
$ curl -i http://127.0.0.1:5000/countries

HTTP/1.0 200 OK
Content-Type: application/json
...

[
 {
 "area": 513120,
 "capital": "Bangkok",
 "id": 1,
 "name": "Thailand"
 },
 {
 "area": 7617930,
 "capital": "Canberra",
 "id": 2,
 "name": "Australia"
 },
 {
 "area": 1010408,
 "capital": "Cairo",
 "id": 3,
 "name": "Egypt"
 },
 {
 "area": 357022,
 "capital": "Berlin",
 "id": 4,
 "name": "Germany"
 }
]
```

This returns the full list of countries in the system, with the newest country at the bottom.

This is just a sampling of what Flask can do. This application could be expanded to include endpoints for all the other HTTP methods. Flask also has a large ecosystem of extensions that provide additional functionality for REST APIs, such as  [database integrations](https://realpython.com/flask-connexion-rest-api/),  [authentication](https://realpython.com/flask-google-login/), and background processing.
