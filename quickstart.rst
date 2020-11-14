Getting Started
========================================

System Requirements:
--------------------

-  Linux or Mac OS X with python 3.5+ installed and pycurl
-  Do not use a virtualenv (or have it custom configured to find
   libcurl

Installation
------------
You can install resttest3 either via the Python Package Index (PyPI) or from source.

.. code:: shell

    pip3 install resttest3

Downloading and installing from source
--------------------------------------
Download the latest version of Resttest3 from PyPI:
https://pypi.org/project/resttest3/#files

You can install it by doing the following,

.. code-block:: shell

    tar xvfz resttest3-0.0.0.tar.gz
    cd resttest3-0.0.0
    python3 setup.py build
    python3 setup.py install

Troubleshooting Installation
----------------------------

Almost all installation issues are due to problems with PyCurl and
PyCurl's native libcurl bindings.
It is easy to check if PyCurl is installed correctly:

.. code:: shell

   python -c 'import pycurl'

If this returns correctly, pycurl is installed, if you see an
ImportError or similar, it isn't.
You may also verify the pyyaml installation as well, since that can
fail to install by pip in rare circumstances.

Error installing by pip
------------------------


``__main__.ConfigurationError: Could not run curl-config: [Errno 2] No such file or directory``

This is caused by libcurl not being installed or recognized: first
install pycurl using native packages as above. Alternately, try
installing just the libcurl libraries:

-  On Ubuntu/Debian:

.. code:: shell

   sudo apt-get install libcurl4-openssl-dev

-  On CentOS/RHEL:

.. code:: shell

    yum install libcurl-devel

Simple Testcase
----------------
This will check that APIs accept operations, and will smoketest an
application

sample.yaml:

.. code-block:: yaml

   - config:
      - testset: "Basic tests"
   - test:
      - name: "Basic test"
      - url: "/api/person/"
   - test:
      - name: "Get single person"
      - url: "/api/person/1/"
   - test:
      - name: "Delete a single person, verify that works"
      - url: "/api/person/1/"
      - method: 'DELETE'
   - test: # create entity by PUT
      - name: "Create/update person"
      - url: "/api/person/1/"
      - method: "PUT"
      - body: '{"first_name": "Gaius","id": 1,"last_name": "Baltar","login": "gbaltar"}'
      - headers: {'Content-Type': 'application/json'}
      - validators:  # This is how we do more complex testing!
        - compare: {header: content-type, comparator: contains, expected:'json'}
        - compare: {jsonpath_mini: 'login', expected: 'gbaltar'}  # JSON extraction
        - compare: {raw_body:"", comparator:contains, expected: 'Baltar' } # Test raw response
   - test: # create entity by POST
      - name: "Create person"
      - url: "/api/person/"
      - method: "POST"
      - body: '{"first_name": "William","last_name": "Adama","login": "theadmiral"}'
      - headers: {Content-Type: application/json}

Running Simple Test
---------------------
To run the above test

.. code-block:: shell

    resttest3 --url http://api.example.com --test sample.yaml

JSON Validation
---------------
Resttest3 use jsonschema validator to validate the response object of the API. A simple set of tests that show how json validation can be used to
check contents of a response.

response_validation.yaml:

.. code-block:: yaml

    - test:
        - url: /api/person/1/
        - validators:
            - json_schema: {schema: {file: 'person_schema.json'}}

person_schema.json

.. code-block:: json

    {
        "$schema": "http://json-schema.org/draft-04/schema#",
        "title": "Person",
        "description": "A person from the example app api",
        "type": "object",
        "required":["id", "first_name"],
        "properties": {
            "id": {
                "type": "integer",
                "description": "Unique person ID"
            },
            "first_name": {
                "type": "string"
            },
            "last_name": {
              "type": "string"
            },
            "login": {
                "type": "string"
            }
        }
    }


The above example ``/api/person/1/`` should expected to return the response like

.. code-block:: json

   {
     "id": 23,
     "first_name": "Abhilash"
   }

because the **required** keyword in the schema force to have the **id** and **first_name** , all other fields are optional

Status Code Test
----------------
Most of the time we may need to test the HTTP status code, For example if you have an API which require
Auth information to get the expected result. If we call the API without providing the auth information
we should get 401 status code

Let's fix that,

.. code-block:: yaml

   - config:
      - testset: "Basic tests"
   - test:
      - name: "Get single person"
      # Group multiple test into one group, So it can share same environment variables
      - group: "Account"
      - url: "/api/person/1/"
      - expected_status: [401] # Here test fail if we did not get status code 401


Basic authentication
--------------------
To do Basic authentication Resttest3 support ``auth_username`` and ``auth_password``

Lets try the above example with basic auth

.. code-block:: yaml

   - config:
      - testset: "Basic tests"
   - test:
      - name: "Get single person"
      - group: "Account"
      - url: "/api/person/1/"
      - headers: {Content-Type: application/json}
      - auth_username: "foobar"
      - auth_password: "secret"
      - expected_status: [200]

OAuth 2.0 Bearer Token Usage
----------------------------

Bearer Tokens are the predominant type of access token used with OAuth 2.0.

A Bearer Token is an opaque string, not intended to have any meaning to clients using it. Some servers will issue tokens that are a short string of hexadecimal characters, while others may use structured tokens.

.. code-block:: yaml

    - config:
        - testset: "Testing the person api"
        - url: &url "api/person/"

    #*********** [TEST-1] *****************************************************
     #Fetching case details
    - test:
        - name: "Fetching person details [TEST-1]"
        - url: *url
        - headers: {'token': 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9'}
        - expected_status: [200]
        - group: "Account"
        - delay: 0
        - validators:
            - json_schema: {schema: {file: './schema/person_get_schema.json'}}
            - compare: {"jsonpath_mini": "data", comparator: "contains", expected: "first_name"}

    #*********** [TEST-2] *****************************************************
     #Passing invalid token
    - test:
        - name: "Passing invalid token [TEST-2]"
        - url: *url
        - headers: {'token': 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9==' }
        - expected_status: [401]
        - group: "Account"
        - delay: 0
        - validators:
            - json_schema: {schema: {file: './schema/error_response.json'}}
            - compare: {"jsonpath_mini": "status", comparator: "str_eq", expected: "FAILURE"}

