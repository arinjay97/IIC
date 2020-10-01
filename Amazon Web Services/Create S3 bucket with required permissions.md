# CREATE S3 BUCKET WITH REQUIRED PERMISSIONS

The requirement of this task was to create a S3 bucket with 
    - Bucket not being publically acessible
    - Objects inside the bucket are publically acessible

A secondary task was to write functions in Python to upload and download objects from the bucket for users with the required permissions.

## To create the bucket

1. Login to the [S3 console](https://s3.console.aws.amazon.com/s3/home) and choose **Create bucket**.

2. Enter a bucket name of your choice and choose **Create bucket** with the default settings. The permissions for the bucket should be as seen below. Only the `Block public and cross-account access to buckets and objects through any public bucket or access point policies` should be [x].

![Create Bucket](/screenshots/Amazon%20Web%20Services/Create%20Bucket.png)

3. Create a folder `home` in the bucket using the console and another folder inside it with a name of your choice.


## Create IAM User and grant required permissions

1. Login to [IAM Management Console](https://console.aws.amazon.com/iam/home) and select **Users** on the left hand menu.

2. Choose **Add User**. Enter a name for the user of your choice (must be same as the name of the folder created inside the bucket) and select Access type as required. Select **Next: Permissions**.

![Add User 1](/screenshots/Amazon%20Web%20Services/Add%20User%201.png)

3. In the next menu, choose **Attach existing policies directly** and select **Create Policy**.

4. Select the JSON editor and put the following in it to give permissions.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowGroupToSeeBucketListInTheConsole",
            "Action": [
                "s3:ListAllMyBuckets",
                "s3:GetBucketLocation"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::*"
            ]
        },
        {
            "Sid": "AllowRootAndHomeListingOfCompanyBucket",
            "Action": [
                "s3:ListBucket"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::examplebucket97"
            ],
            "Condition": {
                "StringEquals": {
                    "s3:prefix": [
                        "",
                        "home/"
                    ],
                    "s3:delimiter": [
                        "/"
                    ]
                }
            }
        },
        {
            "Sid": "AllowListingOfUserFolder",
            "Action": [
                "s3:ListBucket"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::examplebucket97"
            ],
            "Condition": {
                "StringLike": {
                    "s3:prefix": [
                        "home/${aws:username}/*",
                        "home/${aws:username}"
                    ]
                }
            }
        },
        {
            "Sid": "AllowAllS3ActionsInUserFolder",
            "Action": [
                "s3:*"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::examplebucket97/home/${aws:username}/*"
            ]
        },
        {
            "Sid": "DontAllowAllDeleteObjectActionsInUserFolder",
            "Action": [
                "s3:DeleteObject"
            ],
            "Effect": "Deny",
            "Resource": [
                "arn:aws:s3:::examplebucket97/home/${aws:username}/*"
            ]
        }
    ]
}
```

These permissions allow the user to:
    - View the bucket in the S3 console
    - Allow user to list the `home` and `root` directories inside the bucket
    - Allow user to list contents of folder with the same name as them
    - Allow users to upload, view and modify objects stored in the directory with their name
    - Do not allow user to delete any file from their user folder

5. Give a name to the policy created and choose **Create Policy**

## Upload Objects to the bucket using AWS SDK

For using AWS SDK, it is required to setup the credentials of the user to avoid a **Permission Denied** error when executing the upload code. For this create:
    - **Credentials** file with the following content
    ```
    [default]
    aws_access_key_id=ACCESS_KEY
    aws_secret_access_key=SECRET_ACCESS_KEY
    ```
    - **Config** file with the following content
    ```
    [default]
    region=REGION_CODE
    output=OUTPUT
    ```

These need to be stored at:
    -  ~/.aws/credentials for Linux and MacOS
    - C:\Users\USERNAME\.aws\credentials for Windows

Once the setup is complete, it is very simple to upload and download objects from the bucket with the help of AWS SDK.

The following code can be used to *upload* any file to the bucket
```python
import boto3

s3 = boto3.client('s3')
bucket = "<BUCKET NAME>"
file = "<PATH TO FILE BEING UPLOADED"

with open(file, "rb") as f:
    s3.upload_fileobj(f, bucket, f'PATH/TO/S3/BUCKET/FOLDER/{file}', ExtraArgs={'ACL': 'public-read'})

```

This code will upload the file with the designated path to the S3 bucket directory that is being specified. The **ExtraArgs** parameter allows permission control of the object being uploaded. In this case, the uploaded object will be publically accessible, but the bucket itself is not going to be accessible.

Thus the goal of the task is achieved.

Downloading an object from a S3 bucket is also simple with the help of AWS SDK.

The following code can be used to *download* any file to the bucket
```python
import boto3

s3 = boto3.client('s3')
bucket = "<NAME OF BUCKET>"

s3.download_file(bucket, "PATH/TO/OBJECT/IN/BUCKET", "DOWNLOAD LOCATION FOR FILE")
```