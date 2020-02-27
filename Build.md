# Build Your Own Solution

## Running unit tests for customization
* Clone the repository, then make the desired code changes
* Next, run unit tests to make sure added customization passes the tests
```
cd ./deployment
chmod +x ./run-unit-tests.sh  \n
./run-unit-tests.sh \n
```

## Building distributable for customization
* Configure the bucket name of your target Amazon S3 distribution bucket
```
export DIST_OUTPUT_BUCKET=my-bucket-name # bucket where customized code will reside
export SOLUTION_NAME=my-solution-name
export VERSION=my-version # version number for the customized code
```
_Note:_ You would have to create an S3 bucket with the prefix 'my-bucket-name-<aws_region>'; aws_region is where you are testing the customized solution. Also, the assets in bucket should be publicly accessible.

* Now build the distributable:
```
chmod +x ./build-s3-dist.sh \n
./build-s3-dist.sh $DIST_OUTPUT_BUCKET $SOLUTION_NAME $VERSION \n
```

* Deploy the distributable to an Amazon S3 bucket in your account. _Note:_ you must have the AWS Command Line Interface installed.
```
aws s3 cp ./dist/ s3://my-bucket-name-<aws_region>/$SOLUTION_NAME/$VERSION/ --recursive --acl bucket-owner-full-control --profile aws-cred-profile-name \n
```

* Get the link of the solution template uploaded to your Amazon S3 bucket.
* Deploy the solution to your account by launching a new AWS CloudFormation stack using the link of the solution template in Amazon S3.

*** 

## File Structure

```
|-deployment/
  |-build-s3-dist.sh             [ shell script for packaging distribution assets ]
  |-run-unit-tests.sh            [ shell script for executing unit tests ]
  |-solution.yaml                [ solution CloudFormation deployment template ]
|-source/
  |-example-function-js          [ Example microservice function in javascript ]
    |- lib/                      [ Example function libraries ]
  |-example-function-py          [ Example microservice function in python ]

```

Each microservice follows the structure of:

```
|-service-name/
  |-lib/
    |-[service module libraries and unit tests]
  |-index.js [injection point for microservice]
  |-package.json
```

***

## Testing
```bash
# VPC
aws cloudformation create-stack --region cn-northwest-1 \
--stack-name vpc \
--template-body file://01-newvpc.yaml \
--capabilities CAPABILITY_NAMED_IAM \
--parameters \
ParameterKey=NumberOfAZs,ParameterValue=3 \
ParameterKey=AvailabilityZones,ParameterValue=cn-northwest-1a\\,cn-northwest-1b\\,cn-northwest-1c \
--profile zhy

# Security Group
aws cloudformation create-stack --region cn-northwest-1 \
--stack-name sg \
--template-body file://02-securitygroups.yaml \
--capabilities CAPABILITY_NAMED_IAM \
--parameters \
ParameterKey=Vpc,ParameterValue=vpc-085b2b77b51227a8e \
--profile zhy

# RDS
aws cloudformation create-stack --region cn-northwest-1 \
--stack-name rds \
--template-body file://03-rds.yaml \
--capabilities CAPABILITY_NAMED_IAM \
--parameters \
ParameterKey=DatabaseInstanceType,ParameterValue=db.r5.large \
ParameterKey=DatabaseMasterUsername,ParameterValue=root \
ParameterKey=DatabaseMasterPassword,ParameterValue=rootroot \
ParameterKey=DatabaseName,ParameterValue=moodle \
ParameterKey=DatabaseSecurityGroup,ParameterValue=sg-063ed3c8a7bda5f68 \
ParameterKey=NumberOfSubnets,ParameterValue=3 \
ParameterKey=Subnets,ParameterValue=subnet-0feea45882d6bceac\\,subnet-09e25abc30d56c1fb\\,subnet-09f2668334301fb31 \
--profile zhy

# ElastiCache
aws cloudformation create-stack --region cn-northwest-1 \
--stack-name redis \
--template-body file://03-elasticache.yaml \
--capabilities CAPABILITY_NAMED_IAM \
--parameters \
ParameterKey=ElastiCacheClusterName,ParameterValue=moodle \
ParameterKey=ElastiCacheNodeType,ParameterValue=cache.r5.large \
ParameterKey=ElastiCacheSecurityGroup,ParameterValue=sg-02abed66c6dac4d22 \
ParameterKey=NumberOfSubnets,ParameterValue=3 \
ParameterKey=Subnets,ParameterValue=subnet-0feea45882d6bceac\\,subnet-09e25abc30d56c1fb\\,subnet-09f2668334301fb31 \
--profile zhy

# EFS
aws cloudformation create-stack --region cn-northwest-1 \
--stack-name efs \
--template-body file://03-efsfilesystem.yaml \
--capabilities CAPABILITY_NAMED_IAM \
--parameters \
ParameterKey=EC2KeyName,ParameterValue=aws \
ParameterKey=NumberOfSubnets,ParameterValue=3 \
ParameterKey=SecurityGroup,ParameterValue=sg-06a8d9ed5f0ce1045 \
ParameterKey=Subnet,ParameterValue=subnet-0feea45882d6bceac\\,subnet-09e25abc30d56c1fb\\,subnet-09f2668334301fb31 \
ParameterKey=LatestAmiId,ParameterValue=/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 \
--profile zhy

# ALB
aws cloudformation create-stack --region cn-northwest-1 \
--stack-name alb \
--template-body file://03-publicalb.yaml \
--capabilities CAPABILITY_NAMED_IAM \
--parameters \
ParameterKey=Vpc,ParameterValue=vpc-085b2b77b51227a8e \
ParameterKey=PublicAlbAcmCertificate,ParameterValue=arn:aws-cn:iam::057005827724:server-certificate/moodle.demo.lb.joeshi.net \
ParameterKey=PublicAlbSecurityGroup,ParameterValue=sg-0117809d3ddec437f	 \
ParameterKey=NumberOfSubnets,ParameterValue=3 \
ParameterKey=Subnet,ParameterValue=subnet-01a10c46747f1d26f\\,subnet-0e515692ba9700b62\\,subnet-05b4d970835567651	 \
--profile zhy

# CloudFront
aws cloudformation create-stack --region cn-northwest-1 \
--stack-name cloudfront \
--template-body file://04-cloudfront.yaml \
--capabilities CAPABILITY_NAMED_IAM \
--parameters \
ParameterKey=CloudFrontIamCertificateId,ParameterValue=ASCAQ2ROP42GLFTEOGQAL \
ParameterKey=PublicAlbDnsName,ParameterValue=alb-PublicA-8U7UILWL3Q7L-2018506601.cn-northwest-1.elb.amazonaws.com.cn \
ParameterKey=DomainName,ParameterValue=moodle.demo.joeshi.net	\
--profile zhy

# Web
aws cloudformation create-stack --region cn-northwest-1 \
--stack-name web2 \
--template-body file://04-web.yaml \
--capabilities CAPABILITY_NAMED_IAM \
--parameters \
ParameterKey=ElastiCacheClusterEndpointAddress,ParameterValue= \
ParameterKey=DatabaseClusterEndpointAddress,ParameterValue=moodle.cluster-caffyvdhgdo6.rds.cn-northwest-1.amazonaws.com.cn \
ParameterKey=DatabaseMasterUsername,ParameterValue=root \
ParameterKey=DatabaseMasterPassword,ParameterValue=rootroot \
ParameterKey=DatabaseName,ParameterValue=moodle \
ParameterKey=ElasticFileSystem,ParameterValue=fs-117995f4 \
ParameterKey=EC2KeyName,ParameterValue=aws \
ParameterKey=NumberOfSubnets,ParameterValue=3 \
ParameterKey=PublicAlbTargetGroupArn,ParameterValue=arn:aws-cn:elasticloadbalancing:cn-northwest-1:057005827724:targetgroup/PublicALB-vpc-085b2b77b51227a8e/d27713816a00bdcc \
ParameterKey=PublicAlbHostname,ParameterValue=https://alb-PublicA-8U7UILWL3Q7L-2018506601.cn-northwest-1.elb.amazonaws.com.cn \
ParameterKey=Subnet,ParameterValue=subnet-03a02e1b6532e507b\\,subnet-01ad1ea79efdf950c\\,subnet-06ecbb736fdea1a86 \
ParameterKey=WebSecurityGroup,ParameterValue=sg-0a8a4dec1bb667bc7 \
ParameterKey=WebAsgMin,ParameterValue=2 \
ParameterKey=DomainName,ParameterValue=moodle.demo.joeshi.net \
--profile zhy

# ALL
aws cloudformation create-stack --region cn-northwest-1 \
--stack-name Moodle \
--template-url https://joeshi-cn-north-1.s3.cn-north-1.amazonaws.com.cn/moodle-on-aws/dev/00-master.template \
--capabilities CAPABILITY_NAMED_IAM \
--parameters \
ParameterKey=EC2KeyName,ParameterValue=aws \
ParameterKey=DomainName,ParameterValue=moodle.demo.joeshi.net \
ParameterKey=AvailabilityZones,ParameterValue=cn-northwest-1a\\,cn-northwest-1b\\,cn-northwest-1c \
ParameterKey=EfsGrowth,ParameterValue=1 \
ParameterKey=DatabaseMasterUsername,ParameterValue=root \
ParameterKey=DatabaseMasterPassword,ParameterValue=rootroot \
ParameterKey=CloudFrontIamCertificateId,ParameterValue=ASCAQ2ROP42GLFTEOGQAL \
ParameterKey=PublicAlbDomainName,ParameterValue=moodle.demo.lb.joeshi.net \
ParameterKey=PublicAlbCertificateArn,ParameterValue=arn:aws-cn:iam::057005827724:server-certificate/moodle.demo.lb.joeshi.net \
ParameterKey=DeployWithSessionCache,ParameterValue=true \
--profile zhy
```

Copyright 2019 Amazon.com, Inc. or its affiliates. All Rights Reserved.

Licensed under the Apache License Version 2.0 (the "License"). You may not use this file except in compliance with the License. A copy of the License is located at

    http://www.apache.org/licenses/

or in the "license" file accompanying this file. This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, express or implied. See the License for the specific language governing permissions and limitations under the License.
