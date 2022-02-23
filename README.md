
# infrastructure

## aws configure

aws configure --profile=dev

aws configure --profile=demo

export AWS_PROFILE=dev

export AWS_PROFILE=demo

## aws networking

aws cloudformation create-stack --stack-name <stack_name> --template-body <path_to_file>

aws cloudformation delete-stack --stack-name <stack_name>

aws --profile=dev ec2 describe-vpcs //aws ec2 describe-vpcs --filters Name=Name,Values=dev