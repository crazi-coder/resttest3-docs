Validation
==========
Validators test response bodies for correctness. They perform a test on
the response body, with context supplied, and return a value that will
evaluate to boolean True or False.

Optionally, validators can return a Failure which evaluates to False,
but supplies additional information.

extract_test
-----------------------
This run an extractor to extract a value from the body and test it using a function.

-  **Arguments:**

   -  (extractor): an extractor definition, see above, named by
      extractor type
   -  test: a test function to apply, which returns true or false (see
      list below)

**Examples: Test key does not exist**

.. code-block:: yaml

   - validators:
     - extract_test: {jsonpath_mini: "data.not_exist", test:"not_exists"}

**Test Functions**
We have to function to here to check if there's a value exists or not

- exists
- not_exists

Compare
-------
This run an extractor and compare results to expected value

**Special note about encodings:** when testing raw binary bodies
and strings for expected values, the expected value string is
automatically encoded as UTF-8 bytes to allow for comparison. Use
a !!bytes YAML if this is a problem, to avoid implicit encoding.

**Arguments:**

-  (extractor): an extractor definition, see above, named by extractor type
-  comparator: a comparator function to apply, which returns true or false (see list below)
-  expected: *value is*:

   -  expected value (a literal)
   -  a template: ``{template: 'template_string'}`` - gotcha here, you need to use 'str_eq' comparator if you want to template numeric values.
   -  an extractor definition. Yes, you can compare two parts of the response body.

**Examples:**

.. code-block:: yaml

   validators:
   # Check the user name matches
   - compare: {jsonpath_mini: "user_name", comparator: "eq", expected: 'neo'}
   # Check the total_count key has value over 10
   - compare: {jsonpath_mini: "total_count", comparator: "gt", expected: 10}
   # Check the user's login
   -  compare: {jsonpath_mini: "first_name", comparator: "str_eq", expected: 'abhilash'}

Comparator Functions
---------------------
This are the list of ``comparator`` can use.

+------------------------------------+--------------------------------------------------------------+--------------------------------------------------------------------------+
| Name(s)                            | Description                                                  | Details for comparator(A, B)                                             |
+====================================+==============================================================+==========================================================================+
| 'count_eq'                         | Check length of body/str or count of elements equals value   | length(A) == B or -1 if cannot obtain length                             |
+------------------------------------+--------------------------------------------------------------+--------------------------------------------------------------------------+
| 'lt'                               | Less Than                                                    | A < B                                                                    |
+------------------------------------+--------------------------------------------------------------+--------------------------------------------------------------------------+
| 'le'                               | Less Than Or Equal To                                        | A <= B                                                                   |
+------------------------------------+--------------------------------------------------------------+--------------------------------------------------------------------------+
| 'eq'                               | Equals                                                       | A == B                                                                   |
+------------------------------------+--------------------------------------------------------------+--------------------------------------------------------------------------+
| 'str_eq'                           | Values are Equal When Converted to String                    | str(A) == str(B) -- useful for comparing templated numbers/collections   |
+------------------------------------+--------------------------------------------------------------+--------------------------------------------------------------------------+
| 'ne'                               | Not Equals                                                   | A != B                                                                   |
+------------------------------------+--------------------------------------------------------------+--------------------------------------------------------------------------+
| 'ge'                               | Greater Than Or Equal To                                     | A >= B                                                                   |
+------------------------------------+--------------------------------------------------------------+--------------------------------------------------------------------------+
| 'gt'                               | Greater Than                                                 | A > B                                                                    |
+------------------------------------+--------------------------------------------------------------+--------------------------------------------------------------------------+
| 'contains'                         | Contains                                                     | B in A                                                                   |
+------------------------------------+--------------------------------------------------------------+--------------------------------------------------------------------------+
| 'contained_by'                     | Contained By                                                 | A in B                                                                   |
+------------------------------------+--------------------------------------------------------------+--------------------------------------------------------------------------+
| 'type'                             | Type of variable is                                          | A instanceof (at least one of) B                                         |
+------------------------------------+--------------------------------------------------------------+--------------------------------------------------------------------------+
| 'regex'                            | Regex Equals                                                 | A matches regex B                                                        |
+------------------------------------+--------------------------------------------------------------+--------------------------------------------------------------------------+

Types for type test
--------------------
Map of name to types it will match

+---------------+-----------------------------------+
| Type          | Description                       |
+===============+===================================+
| null          | JSON null type                    |
+---------------+-----------------------------------+
| none          | JSON null type                    |
+---------------+-----------------------------------+
|number         | JSON number type(int, long, float)|
+---------------+-----------------------------------+
|int            | JSON number type if not a float   |
+---------------+-----------------------------------+
|float          | JSON number type if a float       |
+---------------+-----------------------------------+
|boolean        | JSON boolean                      |
+---------------+-----------------------------------+
|string         | JSON string                       |
+---------------+-----------------------------------+
|array          | JSON array                        |
+---------------+-----------------------------------+
|list           | JSON array                        |
+---------------+-----------------------------------+
|dict           | JSON object                       |
+---------------+-----------------------------------+
|map            | JSON object                       |
+---------------+-----------------------------------+
|scalar         | JSON any type but object or array |
+---------------+-----------------------------------+
|collection     | JSON array or object              |
+---------------+-----------------------------------+

JSONSchema Validator (Optional)
-------------------------------

**Note:** This requires the 'jsonschema' python module to be installed.
If not installed, you will be unable to use this validator, and an error
message will be printed when tests are run (at "warn" log level).
This validator lets you validate a request against a `JSON Schema <http://json-schema.org/>`__, which can be in the test
body or an external file (as per the request body).

**Arguments:**

-  schema - the JSON schema to use in validating the request body

**Examples:**

Validate against a schema in file 'miniapp-schema.json'

.. code:: yaml

    - test:
        - url: /api/person/1/
        - validators:
            - json_schema: {schema: {file: 'miniapp-schema.json'}}