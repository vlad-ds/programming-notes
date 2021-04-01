# AWS Boto3

AWS SDK for Python. 

Go to AWS IAM and create a user with programmatic access. Give the necessary policies:

* AmazonS3FullAccess
* AmazonSNSFullAccess
* AmazonRekognitionFullAccess
* ComprehendFullAccess

The key and secret identify our user. 

Services: 

* IAM. Identity access management.
* S3. Simple storage service. 
* SNS. Simple notification service. Send emails and texts to alert subscribers. 
* Comprehend. Sentiment analysis on blocks of text. 
* Rekognition. Extracts texts from images.

Here's how to access S3:

```python
# Generate the boto3 client for interacting with S3
s3 = boto3.client('s3', region_name='us-east-1', 
                        # Set up AWS credentials 
                        aws_access_key_id=AWS_KEY_ID, 
                         aws_secret_access_key=AWS_SECRET)
# List the buckets
buckets = s3.list_buckets()

# Print the buckets
print(buckets)
```

S3 and cloud storage. Actions:

* Create bucket
* List buckets
* Delete bucket

```python
import boto3

# Create boto3 client to S3
s3 = boto3.client('s3', region_name='us-east-1', 
                         aws_access_key_id=AWS_KEY_ID, 
                         aws_secret_access_key=AWS_SECRET)

# Create the buckets
response_staging = s3.create_bucket(Bucket='gim-staging')
response_processed = s3.create_bucket(Bucket='gim-processed')
response_test = s3.create_bucket(Bucket='gim-test')

# Print out the response
print(response_staging)
```

```python
# Get the list_buckets response
response = s3.list_buckets()

# Iterate over Buckets from .list_buckets() response
for bucket in response['Buckets']:
  
  	# Print the Name for each bucket
    print(bucket['Name'])
```

```python
# Delete the gim-test bucket
s3.delete_bucket(Bucket='gim-test')

# Get the list_buckets response
response = s3.list_buckets()

# Print each Buckets Name
for bucket in response['Buckets']:
    print(bucket['Name'])
```

The files in S3 are called objects. An object can be anything. Each bucket has a name, each object is identified by a key. A bucket name is just a string. An object's key is the full path from the bucket root. The bucket name must be unique in all S3. The object key must be unique within the bucket. A bucket has many objects, an object can only be in one bucket. 

```python
# Upload final_report.csv to gid-staging
s3.upload_file(Bucket='gid-staging',
              # Set filename and key
               Filename='final_report.csv', 
               Key='2019/final_report_01_01.csv')

# Get object metadata and print it
response = s3.head_object(Bucket='gid-staging', 
                       Key='2019/final_report_01_01.csv')

# Print the size of the uploaded object
print(response['ContentLength'])
```

```python
# List only objects that start with '2018/final_'
response = s3.list_objects(Bucket='gid-staging', 
                           Prefix='2018/final_')

# Iterate over the objects
if 'Contents' in response:
  for obj in response['Contents']:
      # Delete the object
      s3.delete_object(Bucket='gid-staging', Key=obj['Key'])

# Print the keys of remaining objects in the bucket
response = s3.list_objects(Bucket='gid-staging')

for obj in response['Contents']:
  	print(obj['Key'])
```





