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

## Testing

### 1. Install dependencies
```bash
cd ~/ws-serverless-patterns/users
pip install -r requirements.txt
pip install -r ./tests/requirements.txt
```

### 2. Run unit tests
```bash
cd ~/workshop/ws-serverless-patterns/users
python -m pytest tests/unit -v
```

### 3. Run integration tests
```bash
cd ~/ws-serverless-patterns/users
export ENV_STACK_NAME=ws-serverless-patterns-users
python -m pytest tests/integration -v
```

## Cleanup

To avoid incurring future charges, delete the resources created by this project:

```bash
sam delete
```