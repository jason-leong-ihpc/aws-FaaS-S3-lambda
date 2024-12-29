# Assignment CE8 2.12 FaaS - S3 with Lambda
by Jason Leong

## Assignment: Resource Policies
Given a Lambda function that is triggered upon the creation of files in an S3 bucket, answer the following:  
### 1.	What is the purpose of the execution role on the Lambda function?  
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "arn:aws:logs:${var.region}:${var.account_id}:*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": [
                "arn:aws:logs:${var.region}:${var.account_id}:log-group:/aws/lambda/${aws_lambda_function.test_function.function_name}:*"
            ]
        }
    ]
}
```
The execution role defines the permissions required by the Lambda function to access AWS services and resources to create a new CloudWatch log group, create a log stream within a CloudWatch log group and to write log events to a log stream in a CloudWatch log group. These CloudWatch logs help in monitoring, debugging, and tracking the execution of the Lambda function. The policy above was attached to the execution role to allow these functions to be performed.  
### 2.	What is the purpose of the resource-based policy on the Lambda function?  
```
{
  "Version": "2012-10-17",
  "Id": "default",
  "Statement": [
    {
      "Sid": "255945442255_event_permissions_from_${aws_s3_bucket.test_bucket.bucket}_for_${aws_lambda_function.test_function.function_name}",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "lambda:InvokeFunction",
      "Resource": "${aws_lambda_function.test_function.arn}",
      "Condition": {
        "StringEquals": {
          "AWS:SourceAccount": "${var.account_id}"
        },
        "ArnLike": {
          "AWS:SourceArn": "${aws_s3_bucket.test_bucket.arn}"
        }
      }
    }
  ]
}
```
The resource-based policy on a Lambda function allows external entities such as S3 or other AWS services to invoke the Lambda function. Basically, it defines who or what can trigger the Lambda function and under what conditions. In this example, the policy allows an S3 bucket (defined by the ARN arn:aws:s3:::${bucket-name}) to invoke the Lambda function.
### 3.	If the function is needed to upload a file into an S3 bucket, describe (i.e no need for the actual policies)  
   -  What is the needed update on the execution role? 
```  
{  
    "Version": "2012-10-17",  
    "Statement": [  
        {  
            "Effect": "Allow",  
            "Action": [  
                "s3:ListBucket"  
            ],  
            "Resource": [  
                ${aws_s3_bucket.test_bucket.arn}"  
            ]  
        },  
        {  
            "Effect": "Allow",  
            "Action": [  
                "s3:PutObject",  
                "s3:GetObject",  
            ],  
            "Resource": [  
                "${aws_s3_bucket.test_bucket.arn}/*"  
            ]  
        }  
    ]  
}  
```  
For the lambda function to be able to upload a file into an S3 bucket, a policy with appropriate permissions will have to be attached to the role. In the example attached, a policy which allows s3:ListBucket, s3:GetObject and s3:PutObject was attached to the execution role to allow the Lambda function to upload a file into the S3 bucket.
   -  What is the new resource-based policy that needs to be added? To which resource?
No new resource based policy is needed for the Lambda function to be able to create a file into an S3 bucket once the execution role has been updated with the policy with correct permissions as stated earlier. A simple test function can be used to test that the Lambda function is able to create a file into an S3 bucket:  
```
import boto3 

def lambda_handler(event, context): 
  # Define the bucket name as a string
  bucket_name = "your-bucket-name"  # Replace with the actual bucket name
  file_name = 'test_file.txt' 
  file_content = 'test_file.txt test content.' 

  s3 = boto3.client('s3') 
  s3.put_object(Body=file_content, Bucket=bucket_name, Key=file_name) 
  return { 
      'statusCode': 200, 
      'body': 'File uploaded successfully.' 
  }
```

