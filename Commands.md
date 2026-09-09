## Deployment

### First-time guided deployment

```bash
sam deploy --guided --stack-name ws-serverless-patterns-users
```

### Build and deploy the SAM template

```bash
sam build && sam deploy
```

---

## CloudFormation

### Describe stack details

```bash
aws cloudformation describe-stacks \
  --stack-name <stack_name>
```

### Describe stack resources

```bash
aws cloudformation describe-stack-resources \
  --stack-name <stack_name>
```

---

## Lambda

### Generate a sample event for API Gateway
```bash
  sam local generate-event apigateway aws-proxy
```

### Invoke a Lambda function locally using Docker

```bash
sam local invoke \
  -e events/event-get-user-by-id.json \
  -n env.json
```

### Get the function name (Physical ID) using the logical resource name

```bash
aws cloudformation describe-stacks \
  --stack-name "ws-serverless-patterns-users" \
  --query "Stacks[0].Outputs[?OutputKey=='<function_logical_name>'].OutputValue" \
  --output text
```

### Invoke a deployed Lambda function from the AWS CLI

```bash
aws lambda invoke \
  --function-name "<function_name>" \
  --payload "fileb://./events/event-post-user.json" \
  --cli-binary-format raw-in-base64-out \
  response.json
```

---

## API Gateway

### Get the API Gateway endpoint URL

```bash
export API_ENDPOINT=$(aws cloudformation describe-stacks \
  --stack-name <stack_name> \
  --output text \
  --query "Stacks[0].Outputs[?OutputKey=='APIEndpoint'].OutputValue")
```

---

## Amazon Cognito

### Obtain an ID token

```bash
export ID_TOKEN=$(aws cognito-idp initiate-auth \
  --auth-flow USER_PASSWORD_AUTH \
  --client-id 41cgssv4bs312ij4gtf7j5qp0m \
  --auth-parameters USERNAME="<username>",PASSWORD="<password>" \
  --query 'AuthenticationResult.IdToken' \
  --output text)
```

---

## API Testing

### Send a GET request with an Authorization header

```bash
curl $API_ENDPOINT/users \
  -H "Authorization:$ID_TOKEN"
```

### Send a PUT request with an Authorization header

```bash
curl --location \
  --request PUT \
  $API_ENDPOINT/users/b4e82458-e081-70ab-b429-a10e6cfb8596 \
  --data-raw '{"name": "My name is Jonathan"}' \
  -H "Authorization:$ID_TOKEN" \
  -H "Content-Type: application/json"
```