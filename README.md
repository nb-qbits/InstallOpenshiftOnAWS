# InstallOpenshiftOnAWS
Start by setting up your AWS Instances. I am using MAC and brew makes it easy for me (you can find equivalent way to install as per your OS)

Install AWS CLI

$brew install awscli

$aws --version

aws-cli/1.16.30 Python/3.7.0 Darwin/17.6.0 botocore/1.12.20

$ export AWS_ACCESS_KEY_ID=<your aws key>
  
$ export AWS_SECRET_ACCESS_KEY=<your aws secret>
  
CREATE VPC

$aws ec2 create-vpc --cidr-block 10.0.0.0/16 --region=us-east-2

export EC2_VPC_ID=$(aws ec2 describe-vpcs --region=us-east-1 --output=text | grep 10.0.0.0 | awk '{print $7}')

echo $EC2_VPC_ID

$aws ec2 create-tags --resources --region=us-east-1 $EC2_VPC_ID  --tags 'Key=Name,Value=ops-day'

The VPC was created with a huge CIDR block 10.0.0.0/16 in the above example, that provides 65536 IP addresses. You will get CIDR block to use for your subnets.

Establish Internet Connectivity to your VPC

This will allow internet connectivity to the VPC.

Create an internet gateway

$export EC2_INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway --region=us-east-1|grep InternetGatewayId|awk '{print $2}'|sed -e 's/^"//' -e 's/",$//')

$echo $EC2_INTERNET_GATEWAY_ID

Attach internet gateway to VPC that was created in the last section

$aws ec2 attach-internet-gateway --internet-gateway-id --region=us-east-1 $EC2_INTERNET_GATEWAY_ID --vpc-id $EC2_VPC_ID

Create Route Table

export EC2_ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $EC2_VPC_ID --region=us-east-1 | grep RouteTableId | awk '{print $2}'| sed -e 's/^"//' -e 's/",$//')

echo $EC2_ROUTE_TABLE_ID

Create a route in the route table that points all traffic (0.0.0.0/0) to the Internet gateway.

$aws ec2 create-route --route-table-id --region=us-east-1 $EC2_ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $EC2_INTERNET_GATEWAY_ID

{
    "Return": true
}

echo $EC2_INTERNET_GATEWAY_ID

$aws ec2 describe-route-tables --region=us-east-1 --route-table-id $EC2_ROUTE_TABLE_ID

"RouteTables": [
        {
            "Associations": [],
            "PropagatingVgws": [],
            "RouteTableId": "rtb-057ca0c5eef85e9c5",
            "Routes": [
                {
                    "DestinationCidrBlock": "10.0.0.0/16",
                    "GatewayId": "local",
                    "Origin": "CreateRouteTable",
                    "State": "active"
                },
                {
                    "DestinationCidrBlock": "0.0.0.0/0",
                    "GatewayId": "igw-005c937810d0f6819",
                    "Origin": "CreateRoute",
                    "State": "active"
                }
            ],
            "Tags": [],
            "VpcId": "vpc-08a23e9137fecd06d"
        }
    ]
}


