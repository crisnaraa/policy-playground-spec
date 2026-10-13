# Application Design Questions

## Question 1 of 7
How should policy jobs (SQL/Python) be executed in the backend? Choose the execution model:
1. AWS Lambda (serverless, short-lived executions)
2. AWS Batch (managed batch jobs, supports long-running tasks)
3. ECS Task (container-based, ad hoc task execution)
4. Step Functions orchestrating Lambda + ECS (hybrid for complex pipelines)
X) Other — please describe
[Answer]: ECS Task (container-based, ad hoc task execution)

## Question 2 of 7
How should policy versions be integrated from Git into the platform? Choose the ingestion approach:
1. AWS CodePipeline + CodeCommit (native AWS Git + CI/CD)
2. GitHub/GitLab webhook → API Gateway → Lambda ingestion (external Git + AWS backend)
3. S3-based sync (policy files pushed to S3, versioned with S3 Versioning)
4. Manual upload via the UI (no Git integration at runtime)
X) Other — please describe
[Answer]: Git integrado a AWS

## Question 3 of 7
What is the expected volume of concurrent policy simulation sessions? This guides scaling decisions:
1. Small team (< 10 concurrent users)
2. Medium team (10–50 concurrent users)
3. Large organization (50–200 concurrent users)
4. Enterprise scale (200+ concurrent users, multi-region)
X) Other — please describe
[Answer]: Medium team (10–50 concurrent users)

## Question 4 of 7
How should simulation results be surfaced to the user?
1. Real-time streaming (WebSocket / AWS API Gateway WebSocket)
2. Polling (frontend polls a job status endpoint every few seconds)
3. Async notification (SNS/SES email or in-app notification when job completes)
4. Combination: polling for status + WebSocket push when done
X) Other — please describe
[Answer]: Polling (frontend polls a job status endpoint every few seconds)

## Question 5 of 7
Should the platform support collaborative features (multiple analysts working on the same scenario/session)?
1. No — individual sessions only, no real-time collaboration
2. Yes — shared sessions with read-only view for others
3. Yes — full collaborative editing (multiple users in the same session)
4. Not now, but design the architecture to support it in the future
X) Other — please describe
[Answer]: Yes — shared sessions with read-only view for others

## Question 6 of 7
How should the synthetic/anonymized data sandbox be managed?
1. Pre-generated static dataset refreshed periodically (e.g., weekly S3 snapshot)
2. On-demand generation using AWS Glue or a Lambda data generator
3. Always-on dedicated sandbox database (separate RDS/DynamoDB environment)
4. Dynamic masking at query time using AWS Macie + Glue DataBrew
X) Other — please describe
[Answer]: Always-on dedicated sandbox database (separate RDS/DynamoDB environment)

## Question 7 of 7
What is the preferred frontend deployment model?
1. Single Page Application (React/Vue) hosted on S3 + CloudFront
2. Server-side rendered app (Next.js) on ECS/Fargate
3. Micro-frontend architecture (multiple independent SPAs per feature)
4. Internal tool hosted on AWS Amplify
X) Other — please describe
[Answer]: Micro-frontend architecture (multiple independent SPAs per feature)
