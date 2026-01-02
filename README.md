# URL Shortener

A serverless URL shortener with user authentication and ownership tracking, built with AWS Lambda, DynamoDB, and API Gateway.

## Features

- User registration and JWT-based authentication
- Create shortened URLs with random code generation
- URL deduplication per user
- View personal URL collection
- Track click statistics (click count, last clicked)
- Public redirect (no authentication required)
- Delete owned URLs

## Tech Stack

- **AWS Lambda** - Serverless compute (Node.js 24.x)
- **Amazon DynamoDB** - NoSQL database with GSI for user queries
- **API Gateway** - REST API endpoints
- **AWS SAM** - Infrastructure as Code
- **JWT** - Token-based authentication
- **bcrypt** - Password hashing

## API Endpoints
```
POST   /auth/register                     Register new user
POST   /auth/login                        Login and get JWT token
POST   /urls                              Create short URL (authenticated)
GET    /my-urls                           Get user's URLs (authenticated)
GET    /urls/{shortCode}/stats            Get URL statistics (owner only)
DELETE /urls/{shortCode}                  Delete URL (owner only)
GET    /{shortCode}                       Redirect to long URL (public)
```

## Database Schema

**Users Table**
- Primary Key: `userId` (String)
- Attributes: `email`, `passwordHash`, `createdAt`

**URLs Table**
- Primary Key: `shortCode` (String)
- GSI: `UserUrlsIndex` on `userId`
- Attributes: `userId`, `longUrl`, `clickCount`, `createdAt`, `lastClickAt`

## Local Development

Prerequisites: Node.js, Docker, AWS CLI, AWS SAM CLI

### Setup

1. Start DynamoDB Local:
```bash
docker run -p 8000:8000 amazon/dynamodb-local
```

2. Create local tables:
```bash
aws dynamodb create-table --table-name users --attribute-definitions AttributeName=userId,AttributeType=S --key-schema AttributeName=userId,KeyType=HASH --billing-mode PAY_PER_REQUEST --endpoint-url http://localhost:8000

aws dynamodb create-table --table-name urls --attribute-definitions AttributeName=shortCode,AttributeType=S AttributeName=userId,AttributeType=S --key-schema AttributeName=shortCode,KeyType=HASH --global-secondary-indexes IndexName=UserUrlsIndex,KeySchema=[{AttributeName=userId,KeyType=HASH}],Projection={ProjectionType=ALL} --billing-mode PAY_PER_REQUEST --endpoint-url http://localhost:8000
```

3. Build and run:
```bash
sam build
sam local start-api --env-vars env.json
```

## Deployment
```bash
sam build
sam deploy --guided
```

## Project Structure
```
url-shortener/
├── functions/          # Lambda function handlers
│   ├── register/
│   ├── login/
│   ├── create-url/
│   ├── redirect/
│   ├── get-my-urls/
│   ├── get-stats/
│   └── delete-url/
├── shared/             # Shared utilities
│   ├── auth/          # JWT, password utilities
│   ├── db/            # DynamoDB client
│   ├── validators/    # Input validation
│   ├── responses/     # Standard API responses
│   ├── errors/        # Custom error classes
│   └── logger/        # Structured logging
├── template.yaml      # SAM infrastructure template
└── env.json          # Local environment variables
```

## Learning Goals

This project demonstrates:
- Vertical slice development approach
- Shared utilities pattern for Lambda functions
- Production-grade error handling
- JWT authentication implementation
- DynamoDB access patterns with GSI
- Professional development workflow