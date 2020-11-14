Workflow Testing
================

Workflow testing is a type of API testing, which checks that the each api and the related API return the desired outcome.
This usually involves getting the access information such as token or basic auth information and pass over the several stages or steps.
For any business process, testing of these sequential steps is defined as "WorkFlow Testing".

Workflow Testing Example
------------------------
Lets have an example to search github user **crazi-coder** and go over the list of repos

.. code-block:: yaml

    - config:
        - testset: "Testing the github api"
        - variable_binds: {'search_term': 'crazi-coder'}

    - test:
        - name: "search for user crazi-coder"
        - url: {'template': '/search/users?q=$search_term'}
        - expected_status: [200]
        - group: "Github"
        - delay: 0
        - extract_binds:
          - 'repos': {'jsonpath_mini': 'items.0.repos_url'}
        - validators:
            - compare: {"jsonpath_mini": "total_count", comparator: "eq", expected: 1}

    - test:
        - name: "Getting list of repos"
        - url: {'template': '$followers'}
        - absolute_urls: true
        - expected_status: [200]
        - group: "Github"
        - delay: 0


Declare Variable