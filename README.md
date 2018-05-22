## PyMacaron

PyMacaron is a python framework for defining json/REST apis, implementing and deploying them. PyMacaron lets you focus on your API design and endpoint implementation, while taking care of all the rest.

### From API design to live server

Create and deploy a Flask-based REST api running as a Docker container on
amazon AWS Elastic Beanstalk, in 3 steps:

* Write a swagger specification for your api
* Tell which Python method to execute for every swagger endpoint
* Implement the Python methods

BOOM! Your are live on Amazon AWS!

PyMacaron abstracts away all the scaffholding of structuring your Python app,
defining routes, serializing/deserializing between json, Python objects and
databases, containerizing your app and deploying it on Amazon.

What's left in your codebase is the only thing that matters: the business
logic.

### The PyMacaron ecosystem

[pymacaron](https://github.com/pymacaron/pymacaron) uses
[pymacaron-core](https://github.com/pymacaron/pymacaron-core) to
spawn REST apis into a Flask app, based on a swagger
specification describing all API endpoints in a friendly yaml format.

[pymacaron](https://github.com/pymacaron/pymacaron) uses
[pymacaron-deploy](https://github.com/pymacaron/pymacaron-deploy)
to easily deploy the micro service as a Docker container running inside Amazon
Elastic Beanstalk.

[pymacaron](https://github.com/pymacaron/pymacaron) gives
you:

* A best practice auto-scalling setup on Elastic Beanstalk
* Error handling and reporting around your api endpoints (via slack or email)
* Endpoint authentication based on JWT tokens
* Transparent mapping from json and DynamoDB to Python objects
* Automated validation of API data and parameters
* A structured way of blackbox testing your API, integrated in the deploy pipeline
* A production-grade stack (docker/gunicorn/Flask)

### Example

See
[pymacaron-helloworld](https://github.com/pymacaron/pymacaron-helloworld)
for an example of a minimal REST api implemented with pymacaron, and
ready to deploy on docker containers in Amazon EC2.

## Your first server

Install pymacaron:

```
pipenv install pymacaron
```

A REST api microservice built with pymacaron has a directory tree looking like
this:

```
.
├── apis                       # Here you put the swagger specifications both of the apis your
│   └── myservice.yaml         # server is implementing, and optionally of 3rd-party apis used
│   └── sendgrid.yaml          # by your server.
│   └── auth0.yaml             # See pymacaron-core for the supported yaml formats.
|
├── myservice
│   └── api.py                 # Implementation of your api's endpoints
│
├── LICENSE                    # You should always have a licence :-)
├── README.rst                 # and a readme!
|
├── klue-config.yaml           # Settings for pymacaron and pymacaron-deploy
|
├── server.py                  # Code to start your server, see below
|
└── test                       # Standard unitests, executed with nosetests
|   └── test_pep8.py
|
└── testaccept                 # Acceptance tests for your api:
    ├── test_v1_user_login.py  # Black-box tests executed against a running server
    └── test_version.py

```

You start your server by going into the project's root directory and doing:

```bash
python server.py --port 8080
```

Where 'server.py' typically looks like:

```python
import os
import sys
import logging
from flask import Flask
from flask_cors import CORS
from pymacaron import API, letsgo


log = logging.getLogger(__name__)


# WARNING: you must declare the Flask app as shown below, keeping the variable
# name 'app' and the file name 'server.py', since gunicorn is configured to
# lookup the variable 'app' inside the code generated from 'server.py'.

app = Flask(__name__)
CORS(app)
# Here you could add custom routes, etc.


def start(port=80, debug=False):

    # Your swagger api files are under ./apis, but you could have them anywhere
    # else really.

    here = os.path.dirname(os.path.realpath(__file__))
    path_apis = os.path.join(here, "apis")

    # Tell pymacaron to spawn apis inside this Flask app.  Set the
    # server's listening port, whether Flask debug mode is on or not. Other
    # configuration parameters, such as JWT issuer, audience and secret, are
    # fetched from 'klue-config.yaml' or the environment variables it refers to.

    api = API(
        app,
        port=port,
        debug=debug,
    )

    # Find all swagger files and load them into pymacaron-core

    api.load_apis(path_apis)

    # Optionally, publish the apis' specifications under the /doc/<api-name>
    # endpoints, so you may open them in Swagger-UI:
    # api.publish_apis()

    # Start the Flask app and serve all endpoints defined in
    # apis/myservice.yaml

    api.start(serve="myservice")


# Entrypoint
letsgo(__name__, callback=start)
```


You run acceptance tests against the above server (started in a separate
terminal) like this:

```bash
cd projectroot
run_acceptance_tests --local
```

You deploy your api to Amazon Elasticbean like this:

```bash
deploy_pipeline --push --deploy
```


## Bootstraping example

Bootstrap your project by cloning [pymacaron-helloworld](https://github.com/pymacaron/pymacaron-helloworld).

## Pluggable features

[pymacaron](https://github.com/pymacaron/pymacaron) in itself lets you
just define an API server and run it locally. You may use additional features
by installing the following extra modules:

### Asynchronous task execution

Install [pymacaron-async](https://github.com/pymacaron/pymacaron-async) by
following [these instructions](https://github.com/pymacaron/pymacaron-async#setup).

### Deploying as a container in Amazon Beanstalk

Install [pymacaron-deploy](https://github.com/pymacaron/pymacaron-deploy) by
following [these instructions](https://github.com/pymacaron/pymacaron-deploy#setup).

### Use Klue's own testing framework

A [convenient library](https://github.com/pymacaron/klue-unit) for black-box testing your API endpoints.

## [Deep Dive](DEEP_DIVE.md)
