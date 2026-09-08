First guided deploy
sam deploy --guided --stack-name ws-serverless-patterns-users

Build and deploy SAM template
sam build && sam deploy

Describe cloudformation stack details
aws cloudformation describe-stacks --stack-name <stack_name>

Describe cloudformation stack resources
aws cloudformation describe-stack-resources --stack-name <stack_name>

Invoke lambda function locally with docker
sam local invoke -e events/event-get-user-by-id.json -n env.json

Get function name (Physical ID) with template logical name
aws cloudformation describe-stacks --stack-name "ws-serverless-patterns-users" --query "Stacks[0].Outputs[?OutputKey=='<function_logical_name>'].OutputValue" --output text

Invoke deployed lambda function from CLI
aws lambda invoke --function-name "<function_name>" --payload "fileb://./events/event-post-user.json"   --cli-binary-format raw-in-base64-out response.json

Get API Gateway URL
export API_ENDPOINT=$(aws cloudformation describe-stacks --stack-name <stack_name> --output text --query "Stacks[0].Outputs[?OutputKey=='APIEndpoint'].OutputValue")

Get cognito TOKEN ID
export ID_TOKEN=$(aws cognito-idp initiate-auth --auth-flow USER_PASSWORD_AUTH --client-id 41cgssv4bs312ij4gtf7j5qp0m --auth-parameters USERNAME="<username>",PASSWORD="<password>" --query 'AuthenticationResult.IdToken' --output text)

Send GET request with authorization header
curl $API_ENDPOINT/users -H "Authorization:$ID_TOKEN"

Send PUT request with authorization header
curl --location --request PUT $API_ENDPOINT/users/b4e82458-e081-70ab-b429-a10e6cfb8596 --data-raw '{"name": "My name is Jonathan"}' -H "Authorization:$ID_TOKEN" -H "Content-Type: application/json"