---
date: "2023-03-08"
title: "AWS DynamoDB CRUD with Boto3"
tags: ["aws", "dynamodb", "boto3", "terraform"]
author: "masoudkf"
description: "We create a DynamoDB table with Terraform, and perform CRUD operations using the AWS SDK for Python (Boto3)."
---

## DynamoDB 
DynamoDB is a fully managed, high-performance, NoSQL database service provided by AWS. It is designed to provide fast and predictable performance with seamless scalability. DynamoDB is a non-relational database that allows users to store and retrieve data, while maintaining low latency and high availability. DynamoDB is a schema-less database, which means that users can store data in any format without defining a schema beforehand. 

DynamoDB stores data in tables. All tables must either have a Primary Key (also known as Partition Key), or a Primary Key and a Sort Key (also known as composite key). In a table that has only a partition key, no two items can have the same partition key value. In a table that has a partition key and a sort key, it's possible for multiple items to have the same partition key value. However, those items must have different sort key values.

Perhaps the most important limitation of DynamoDB is that no item in a table can exceed 400KB. If you have a use case that requires storing items bigger than 400KB, you may want to use a relational database offering from AWS, such as MySQL with AWS RDS, or use a more creative approach of DynamoDB + S3, where you store the index of a file in DynamoDB (which is not going to exceed 400KB) and the data itself in S3 where any individual file can be up to 5TB.

### DynamoDB Data Types
- **Scalar Types**: A scalar type can represent exactly one value. The scalar types are number, string, binary, Boolean, and null.

- **Document Types**: A document type can represent a complex structure with nested attributes, such as you would find in a JSON document. The document types are list and map.

### Create a DynamoDB Table with Terraform

{{< gist masoudkarimif b0738cc57bcbf558970aaa77bd118d36>}}


## Boto3
AWS [Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) is a Python software development kit (SDK) that enables developers to write Python scripts and applications to interact with various AWS services programmatically. Boto3 allows developers to use Python code to interact with AWS resources instead of logging into the AWS Management Console or using command-line tools.

### Boto3 and DynamoDB
You can use Boto3 to interact with DynamoDB. Here's an example of CRUD (Create, Read, Update, and Delete) operations on the `students` table we created earlier:

`main.py`
```python
import boto3

# create a dynamodb resource
dynamodb_resource = boto3.resource("dynamodb")
# create a dynamodb table object
table = dynamodb_resource.Table("students")


def add_item(student):
    # read the docs: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/table/put_item.html
    return table.put_item(Item=student)


def get_item(student_id):
    # read the docs: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/table/get_item.html
    response = table.get_item(Key={"student_id": student_id})
    item = response["Item"]
    return item


def update_item(student):
    # read the docs: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/table/update_item.html
    return table.update_item(
        Key={
            "student_id": student["student_id"],
        },
        UpdateExpression="SET grades = :grades",
        ExpressionAttributeValues={":grades": student["grades"]},
    )


def delete_item(student_id):
    # read the docs: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/table/delete_item.html
    return table.delete_item(
        Key={
            "student_id": student_id,
        }
    )
```

And some tests to make sure our functions work properly:

`test_main.py`
```python
from main import *
import time

# item for testing
item = {
    "student_id": 3098390,
    "first_name": "Alice",
    "last_name": "Wonderland",
    "grades": [90, 85, 100, 94],
}


# run pytest -v -k 'test_add' to run this specific test
def test_add_item():
    res = add_item(item)
    assert res is not None


# run pytest -v -k 'test_get' to run this specific test
def test_get_item():
    res = get_item(item["student_id"])
    assert res is not None
    assert res["first_name"] == item["first_name"]
    assert res["last_name"] == item["last_name"]
    assert res["grades"] == item["grades"]


# run pytest -v -k 'test_update' to run this specific test
def test_update_item():
    updated_item = {"student_id": 3098390, "grades": [90, 100, 100, 100]}
    res = update_item(updated_item)
    assert res is not None
    # sleep for 1 second to make sure the updated record is consistent across all AZs
    time.sleep(1)
    res = get_item(updated_item["student_id"])
    assert res["grades"] == updated_item["grades"]


# run pytest -v -k 'test_delete' to run this specific test
def test_delete_item():
    res = delete_item(item["student_id"])
    assert res is not None

```

To run the test cases using `pytest`, first make sure you have `pytest` installed (`pip install pytest`). You can then use `pytest -v` to run all the tests in the order they are declared in the file. The gotcha here is that because you're interacting with AWS, you need to have proper credentials to run the tests. [AWS Vault](https://github.com/99designs/aws-vault) to the rescue:

```bash
aws-vault exec <profile-name> -- pytest -v
```

