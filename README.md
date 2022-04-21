# Aravind Polepeddi NUID:002102800 infrastucture

# infrastructure

## aws configure

aws configure --profile=dev

aws configure --profile=demo

export AWS_PROFILE=dev

export AWS_PROFILE=demo

## aws cloudformation

aws cloudformation create-stack --stack-name <stack_name> --template-body <path_to_file> --parameters <path_to_file>
//aws cloudformation create-stack --stack-name vpc --capabilities CAPABILITY_NAMED_IAM  --profile=demo --template-body file://csye6225-infra.yml --parameters file://config.json

aws cloudformation delete-stack --stack-name <stack_name>
// aws cloudformation delete-stack --stack-name thur-vpc2 

aws --profile=dev ec2 describe-vpcs //aws ec2 describe-vpcs --filters Name=Name,Values=dev

# to upload cert
$ aws acm import-certificate --certificate file://Certificate.pem --certificate-chain file://CertificateChain.pem --private-key file://PrivateKey.pem --profile=demo 

aws acm import-certificate --certificate fileb://prod_polepeddiaravind.me/prod_polepeddiaravind_me.crt --certificate-chain fileb://prod_polepeddiaravind.me/prod_polepeddiaravind_me.ca-bundle --private-key fileb://prod_polepeddiaravind.me/private.key

