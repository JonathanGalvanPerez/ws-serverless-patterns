# AWS Serverless JWT Authorizer API

This repository contains an AWS Serverless Application Model (SAM) project that demonstrates how to build a secure REST API using Amazon API Gateway, AWS Lambda, Amazon DynamoDB, and Amazon Cognito.

The project features a **Custom Lambda Authorizer** that validates JSON Web Tokens (JWT) issued by an Amazon Cognito User Pool and implements Role-Based Access Control (RBAC) using Cognito User Groups.

## Architecture Overview

This serverless application consists of the following components:

*   **Amazon API Gateway:** Exposes the RESTful endpoints for the application and triggers the custom authorizer for incoming requests.
*   **Amazon Cognito (User Pool):** Handles user registration, authentication, and generates JWT tokens (ID and Access tokens). It includes a specific group for API Administrators.
*   **AWS Lambda (Custom Authorizer - `authorizer.py`):** Intercepts requests from API Gateway, verifies the JWT signature (using JWKS), checks token expiration, and dynamically generates an IAM policy. Regular users are granted access only to their specific user ID routes, while users in the Admin group are granted access to all user routes.
*   **AWS Lambda (API Monolith - `user.py`):** A single Lambda function that acts as a monolith, routing incoming HTTP methods and paths to the appropriate CRUD operations (Create, Read, Update, Delete) for users.
*   **Amazon DynamoDB:** A NoSQL database table (`UsersTable`) used to store the user data.

## Project Structure

```text
.
├── template.yaml               # AWS SAM infrastructure-as-code template
├── samconfig.toml              # SAM CLI configuration file for deployment
├── requirements.txt            # Python dependencies for the Lambda functions
├── README.md                   # This file
├── Commands.md                 # Useful AWS CLI and SAM commands for deployment and testing
├── src/
│   └── api/
│       ├── authorizer.py       # Custom Lambda Authorizer logic (JWT validation & IAM policy generation)
│       └── user.py             # Monolithic API handler for all /users CRUD operations
├── events/                     # Sample event payloads for local testing
└── tests/                      # Unit tests for the Lambda functions
```

## Prerequisites

To deploy and interact with this project, you will need:

*   [AWS CLI](https://aws.amazon.com/cli/) installed and configured with your AWS credentials.
*   [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html) installed.
*   [Python 3.12](https://www.python.org/downloads/) installed locally.

## Deployment

1. **Build the application:** Use the SAM CLI to build the serverless application. This will resolve dependencies and prepare the deployment artifacts.
   ```bash
   sam build
   ```

2. **Deploy the application:** Run the guided deployment process.
   ```bash
   sam deploy --guided
   ```
   *Follow the prompts:*
   *   **Stack Name:** Provide a name for your stack (e.g., `jwt-authorizer-api`).
   *   **AWS Region:** Select your preferred region (e.g., `us-east-1`).
   *   **Parameter UserPoolAdminGroupName:** You can leave the default (`apiAdmins`) or set a custom group name.
   *   Accept the default permissions and allow SAM to create the necessary IAM roles.

3. **Note the Outputs:** Once the deployment finishes, SAM will print an `Outputs` table. Note down the following values as you will need them to test the API:
   *   `APIEndpoint`
   *   `UserPoolId`
   *   `UserPoolClient`
   *   `CognitoAuthCommand`

## Testing the API

### 1. Create a User in Cognito
You can use the AWS CLI or the Cognito AWS Console to create a user. To do it via CLI:
```bash
aws cognito-idp sign-up \
  --client-id <Your-UserPoolClient-Id> \
  --username testuser@example.com \
  --password "MyStrongPassword123!"
```

Confirm the user:
```bash
aws cognito-idp admin-confirm-sign-up \
  --user-pool-id <Your-UserPoolId> \
  --username testuser@example.com
```

### 2. Get a JWT Token
Use the `CognitoAuthCommand` provided in your SAM deployment outputs to retrieve an ID Token. Alternatively, use this command:
```bash
aws cognito-idp initiate-auth \
  --auth-flow USER_PASSWORD_AUTH \
  --client-id <Your-UserPoolClient-Id> \
  --auth-parameters USERNAME=testuser@example.com,PASSWORD="MyStrongPassword123!" \
  --query 'AuthenticationResult.IdToken' \
  --output text
```
*Copy the resulting JWT token string.*

### 3. Make API Requests
Use a tool like `curl` or Postman to interact with your API. You must pass the JWT token in the `Authorization` header.

**Create a user record (POST):**
*(Note: Because of RBAC, a standard user might only be able to interact with endpoints containing their Cognito `sub` as the `{userid}`, unless they are added to the admin group).*
```bash
curl -X POST <Your-APIEndpoint>/users \
  -H "Authorization: <Your-JWT-Token>" \
  -H "Content-Type: application/json" \
  -d '{"name": "John Doe", "email": "johndoe@example.com"}'
```

**Get all users (GET - Requires Admin Privileges):**
```bash
curl -X GET <Your-APIEndpoint>/users \
  -H "Authorization: <Your-JWT-Token>"
```
*If your user is not in the `apiAdmins` Cognito group, this request will return a `403 Forbidden`.*

### 4. Granting Admin Access
To test the administrator routing rules in the authorizer, add your test user to the `apiAdmins` group:

```bash
aws cognito-idp admin-add-user-to-group \
  --user-pool-id <Your-UserPoolId> \
  --username testuser@example.com \
  --group-name apiAdmins
```
*Note: You will need to re-authenticate (Step 2) to get a new JWT token that includes the updated group claims before making admin API requests.*

## Cleanup

To avoid incurring future charges, delete the resources created by this project:

```bash
sam delete
```