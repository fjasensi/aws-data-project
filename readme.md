# AWS Data Analytics Sideproject

This project implements a complete data analytics system using AWS services:

- Data ingestion with Python to S3
- Processing with containerized Lambdas
- Storage in Redshift
- REST APIs using dockerized Lambdas
- Infrastructure as code with CloudFormation
- Automated CI/CD with GitHub Actions (integration with dev and main branches)

## System Architecture

![AWS Data Analytics Architecture](/img/architecture_diagram.png)

The data flow works as follows:

1. The ingestion script collects data and uploads it to S3
2. A containerized Lambda is triggered when new data arrives in S3
3. This Lambda processes the data and loads it into Redshift
4. Additional Lambdas provide APIs to query the data from Redshift

## Project Structure

```
aws-data-project/
├── .github/
│   └── workflows/
│       ├── pr-checks.yml        # GitHub Actions for PR validation
│       ├── deploy-dev.yml       # Workflow for DEV deployment
│       └── deploy-prod.yml      # Workflow for PROD deployment
├── infrastructure/
│   ├── template.yaml            # Main CloudFormation template
│   ├── parameters-dev.json      # Parameters for DEV environment
│   └── parameters-prod.json     # Parameters for PROD environment
├── lambda/
│   ├── common/                  # Shared code between lambdas
│   │   └── db/
│   │       └── redshift.py      # Utilities for Redshift connection
│   ├── ingest/                  # Data ingestion Lambda
│   │   ├── Dockerfile
│   │   ├── app.py
│   │   └── requirements.txt
│   ├── process/                 # Processing Lambda
│   │   ├── Dockerfile
│   │   ├── app.py
│   │   └── requirements.txt
│   └── api/                     # API Lambda
│       ├── Dockerfile
│       ├── app.py
│       └── requirements.txt
├── scripts/
│   └── data_ingest.py           # Script for manual ingestion
├── tests/                       # Unit and integration tests
│   ├── test_ingest.py
│   ├── test_process.py
│   └── test_api.py
├── docker-compose.yml           # For local testing
├── requirements.txt             # Project dependencies
├── buildspec.yml                # CodeBuild configuration
└── README.md                    # Main documentation
```

## Environments

The project supports two environments:

- **DEV**: Automatic deployment when PRs are merged to the `dev` branch
- **PROD**: Automatic deployment when merged from `dev` to `main`

## Prerequisites

- AWS account with permissions to create CloudFormation resources
- Docker installed (for local development)
- Python 3.9+
- AWS CLI configured
- GitHub repository

## Initial Setup

1. Clone this repository:
   
   ```
   git clone https://github.com/your-username/aws-data-project.git
   cd aws-data-project
   ```

2. Create a virtual environment and install dependencies:
   
   ```
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. Configure environment variables in GitHub:
   
   - AWS_ACCESS_KEY_ID
   - AWS_SECRET_ACCESS_KEY
   - AWS_REGION

4. Deploy the initial CloudFormation infrastructure:
   
   ```
   aws cloudformation deploy \
     --template-file infrastructure/template.yaml \
     --stack-name aws-data-project-bootstrap \
     --parameter-overrides Environment=dev \
     --capabilities CAPABILITY_IAM
   ```

## Local Development

### Testing Dockerized Lambdas Locally

1. Build the images:
   
   ```
   docker-compose build
   ```

2. Run lambdas locally:
   
   ```
   docker-compose up process-lambda
   ```

3. Run tests:
   
   ```
   pytest tests/
   ```

## Workflow

The project workflow follows the GitFlow model:

1. Create a feature branch from `dev`:
   
   ```
   git checkout dev
   git pull
   git checkout -b feature/new-functionality
   ```

2. Develop and test locally.

3. Commit and push:
   
   ```
   git add .
   git commit -m "Implement new functionality"
   git push -u origin feature/new-functionality
   ```

4. Create a Pull Request to `dev` in GitHub.
   
   - Automated validations will run in GitHub Actions
   - Once approved and merged, it will automatically deploy to DEV

5. To deploy to production, create a Pull Request from `dev` to `main`.
   
   - Once approved and merged, it will automatically deploy to PROD

This flow ensures that changes are tested first in DEV before reaching PROD.

## Technical Details

### Containerized Lambdas

The Lambdas use Docker containers for greater flexibility. Advantages:

- Complete control over the execution environment
- Use of complex/binary dependencies
- Reuse of existing Docker images
- Better cold start performance (depending on the case)

### Data Ingestion

The ingestion script obtains data from external sources and stores it in S3 with the following pattern:

```
s3://bucket-name/raw/source_name/YYYY/MM/DD/data_TIMESTAMP.parquet
```

### Processing

The processing lambda is triggered by S3 events, transforms the data and loads it into Redshift using the appropriate partitioning strategy.

### API

The API provides endpoints for:

- Historical data queries
- Filtering by date/source
- Aggregations and statistics mediante eventos de S3, transforma los datos y los carga en Redshift usando la estrategia de particionamiento adecuada.

## Monitoring and Maintenance

- Logs are sent to CloudWatch
- Alarms are configured for error in Lambdas
- Infrastructure is updated via CI/CD pipeline
