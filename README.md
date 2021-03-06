# Ansible Cloud Modules for AWS Lambda
#### Version 0.3 [![Build Status](https://travis-ci.org/pjodouin/ansible-lambda.svg)](https://travis-ci.org/pjodouin/ansible-lambda)

These modules help manage AWS Lambda resources including code, configuration, aliases, versions, event source mappings and policy permissions. A specialized s3
module is also provided to help manage s3 event notifications that trigger Lambda functions.
## Requirements
- ansible
- boto3
- importlib (only for running tests on < python 2.7)

## Modules
### lambda_facts:
___
Gathers facts related to AWS Lambda functions.

##### Example Command
`> ansible localhost -m lambda_facts`

##### Example Playbook
```yaml
# Simple example of listing all info for a function
- name: List all for a specific function
  lambda_facts:
    query: all
    function_name: myFunction
  register: my_function_details
# List all versions of a function
- name: List function versions
  lambda_facts:
    query: versions
    function_name: myFunction
  register: my_function_versions
- name: show Lambda facts
  debug: var=Versions
```


### lambda:
___
Add, Update or Delete Lambda related resources. These include 'alias', 'code', 'config', 'mapping', 'policy' and 'version.'

##### Example Playbook
```yaml
- hosts: localhost
  gather_facts: no
  vars:
    state: present
  tasks:
  - name: create a function
    lambda:
      state: "{{ state | default('present') }}"
      type: code
      function_name: myFunction
      runtime: python2.7
      code:
        s3_bucket: myBucket
        s3_key: lambda_packages/lambda.zip
      timeout: 3
      handler: lambda.handler
      role: arn:aws:iam::myAccount:role/someAPI2LambdaExecRole
      description: Another lambda function
      publish: False
  - name: display stuff
    debug: var=results

```

### lambda_invoke:
___
Use to invoke a specific Lambda function.

##### Example Command
`> ansible localhost -m lambda_invoke -a"function_name=myFunction"`


### lambda_s3_event:
___
Add or delete an s3 event notification that calls a lambda function.

##### Example Playbook
```yaml
- hosts: localhost
  gather_facts: no
  vars:
    state: present
    bucket: myBucketName
  tasks:
  - name: add s3 event notifications that trigger a lambda function
    lambda_s3_event:
      state: "{{ state | default('present') }}"
      bucket: "{{ bucket }}"
      lambda_function_configurations:
      - id: lambda-package-myFunction-dev
        lambda_function_arn: arn:aws:lambda:us-east-1:myAccount:function:myFunction:Dev
        events: [ 's3:ObjectCreated:*' ]
        filter:
          key:
            filter_rules:
             - name: prefix
               value: 'dev/'
      - id: lambda-package-myFunction-qa
        lambda_function_arn: arn:aws:lambda:us-east-1:myAccount:function:myFunction:QA
        events: [ 's3:ObjectCreated:*' ]
        filter:
          key:
            filter_rules:
             - name: prefix
               value: 'qa/'
  - name: display stuff
    debug: var=results
```

## Installation

Do the following to install the lambda modules in your Ansible environment:

1. Clone this repository or download the ZIP file.

2. Copy the *.py modules to your installation custom module directory (usually /etc/ansible/modules).

3. Make sure boto3 is installed.








