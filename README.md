
# Quality Movie Data Analysis Project

## 📋 Project Overview

An end-to-end **serverless data engineering pipeline** on AWS that automates movie data ingestion, quality validation, transformation, and analytics. The pipeline is event-driven, ensuring real-time processing of movie datasets with automated data quality checks and intelligent routing of good/bad data.

---

## 🏗️ Architecture

The project implements a fully automated ETL pipeline orchestrated by AWS Step Functions, triggered by S3 file uploads via EventBridge rules.

### Data Flow:
1. **Ingestion Layer**: CSV files land in S3 bucket (`movies-gds/input_data/`)
2. **Event Trigger**: EventBridge detects file creation and triggers Step Functions
3. **Metadata Discovery**: AWS Glue Crawlers catalog metadata for both S3 data and Redshift tables
4. **Data Quality Layer**: AWS Glue Data Quality validates data against business rules (e.g., IMDb rating ranges)
5. **Transformation Layer**: AWS Glue ETL job processes data:
   - **Bad Data**: Routed to S3 quarantine location
   - **Good Data**: Transformed and loaded into Amazon Redshift
6. **Serving Layer**: Redshift materialized views enable fast analytics queries
7. **Notification Layer**: SNS sends email notifications for success/failure scenarios
8. **Orchestration**: Step Functions coordinates the entire workflow

---

## 🛠️ AWS Services Used

| Service | Purpose |
|---------|---------|
| **Amazon S3** | Raw data storage and bad data quarantine |
| **AWS Glue Crawler** | Automatic schema discovery for S3 and Redshift |
| **AWS Glue Data Quality** | Validate data against business rules (IMDb ratings, completeness) |
| **AWS Glue ETL** | Data transformation and loading pipeline |
| **AWS Glue Data Catalog** | Centralized metadata repository |
| **Amazon EventBridge** | Event-driven automation trigger |
| **AWS Step Functions** | Workflow orchestration and state management |
| **Amazon SNS** | Email notifications for pipeline status |
| **Amazon Redshift** | Data warehouse with materialized views for analytics |

---

## 🎯 Key Features

### 1. **Event-Driven Architecture**
- Automatic pipeline trigger when CSV files land in S3
- Zero manual intervention required
- Real-time processing capabilities

### 2. **Data Quality Framework**
- Automated validation using AWS Glue Data Quality
- Business rules enforcement (IMDb rating ranges, data completeness)
- Intelligent data routing:
  - ✅ **Good data** → Transformed and loaded to Redshift
  - ❌ **Bad data** → Quarantined in separate S3 location for review

### 3. **Robust Orchestration**
- Step Functions manages complex workflow dependencies
- Error handling with automatic notifications
- State management across multiple AWS services

### 4. **Monitoring & Alerting**
- SNS email notifications for:
  - ✅ Successful pipeline completion
  - ❌ Quality check failures
  - ❌ ETL job failures
- Complete audit trail through Step Functions execution history

### 5. **Optimized Analytics**
- Redshift materialized views for fast query performance
- Structured data catalog for easy discovery
- Scalable data warehouse architecture

---

## 📊 Data Quality Rules

The pipeline implements the following data quality checks:

- **IMDb Rating Range**: Validates ratings are within acceptable bounds (e.g., 0-10)
- **Completeness Checks**: Ensures required fields are populated
- **Data Type Validation**: Verifies data types match expected schema
- **Null Value Detection**: Identifies and handles missing critical data

---

## 🔄 Pipeline Workflow

```
1. CSV File Upload to S3 (movies-gds/input_data/)
          ↓
2. EventBridge Rule Detects S3 Event
          ↓
3. Step Functions Execution Starts
          ↓
4. Glue Crawler - Scan S3 Data (Metadata Discovery)
          ↓
5. Glue Crawler - Scan Redshift Table (Schema Validation)
          ↓
6. AWS Glue Data Quality Evaluation
          ↓
    ┌─────┴─────┐
    ↓           ↓
Bad Data    Good Data
    ↓           ↓
S3 Quarantine  ETL Transformation
    ↓           ↓
SNS Alert   Load to Redshift
            ↓
         Materialized Views
            ↓
         SNS Success Alert
```

---

## 📁 Project Structure

```
movies-gds/
├── input_data/              # Raw CSV files landing zone
├── bad_data/                # Quarantined data that failed quality checks
└── archive/                 # Processed files archive

Redshift:
├── movie_data_table         # Main data warehouse table
└── materialized_views/      # Pre-computed analytics views
```

---

## 🚀 Setup Instructions

### Prerequisites
- AWS Account with appropriate permissions
- S3 bucket created (`movies-gds`)
- Redshift cluster provisioned
- IAM roles configured for Glue, Step Functions, and EventBridge

### Step 1: Enable EventBridge on S3 Bucket
```bash
# Navigate to S3 Console
# Select bucket: movies-gds
# Properties → Amazon EventBridge → Enable
```

### Step 2: Create Glue Crawlers
```bash
# Crawler 1: S3 Data Crawler
Name: crawl-movies-in-s3
Data source: s3://movies-gds/input_data/
Database: movie_database

# Crawler 2: Redshift Table Crawler
Name: crawl_movies_data_in_redshift
Data source: Redshift connection
Database: movie_database
```

### Step 3: Configure Data Quality Rules
```yaml
# Glue Data Quality Ruleset
Rules:
  - "RowCount > 0"
  - "ColumnValues 'imdb_rating' between 7.5 and 10.0"
  - "Completeness 'movie_title' > 0.95"
  - "Uniqueness 'movie_id' > 0.99"
```

### Step 4: Create Glue ETL Job
```python
# ETL Job Configuration
Name: movie_data_analysis
Type: Spark
Script location: s3://movies-gds/scripts/
Target: Redshift catalog
Bad records path: s3://movies-gds/bad_data/
```

### Step 5: Configure SNS Topics
```bash
# Success Topic
Name: movies-pipeline-success
Email subscription: your-email@example.com

# Failure Topic
Name: movies-pipeline-failure
Email subscription: your-email@example.com
```

### Step 6: Create Step Functions State Machine
```json
{
  "Comment": "Movie Data Processing Pipeline",
  "StartAt": "StartCrawler",
  "States": {
    "StartCrawler": { ... },
    "GetCrawler": { ... },
    "GlueStartJobRun": { ... },
    "SNSPublish": { ... }
  }
}
```

### Step 7: Create EventBridge Rule
```json
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"],
  "detail": {
    "bucket": {
      "name": ["movies-gds"]
    },
    "object": {
      "key": [{
        "prefix": "input_data/"
      }]
    }
  }
}
```

---

## 📈 Usage

### Trigger the Pipeline
Simply upload a CSV file to the S3 bucket:
```bash
aws s3 cp movie_data.csv s3://movies-gds/input_data/
```

The pipeline will automatically:
1. Detect the file upload
2. Trigger Step Functions workflow
3. Run crawlers for metadata discovery
4. Execute data quality checks
5. Process and load data to Redshift
6. Send email notification on completion

### Query Analytics in Redshift
```sql
-- Query materialized views for fast analytics
SELECT * FROM mv_top_rated_movies;
SELECT * FROM mv_genre_statistics;
SELECT * FROM mv_yearly_trends;
```

---

## 🎓 Key Learnings

1. **Event-Driven Architecture**: Implemented serverless, event-driven data pipelines using EventBridge
2. **Data Quality Engineering**: Applied automated data quality frameworks to ensure data reliability
3. **Workflow Orchestration**: Managed complex dependencies using Step Functions state machines
4. **Error Handling**: Built robust error handling with automatic notifications and data quarantine
5. **Cost Optimization**: Utilized serverless services to minimize infrastructure costs
6. **Monitoring**: Implemented comprehensive logging and alerting mechanisms

---

## 🔒 Security Best Practices

- IAM roles with least privilege access
- S3 bucket encryption enabled
- Redshift cluster in private subnet
- VPC endpoints for secure AWS service communication
- Data quality rules prevent data injection attacks

---

## 💰 Cost Optimization

- **Serverless Architecture**: Pay only for actual usage (no idle resources)
- **Glue DPU Optimization**: Right-sized Spark executors
- **Redshift Materialized Views**: Pre-computed queries reduce compute costs
- **S3 Lifecycle Policies**: Automatic archival of processed files

---

## 🐛 Troubleshooting

### Pipeline Not Triggering
- Verify EventBridge is enabled on S3 bucket
- Check EventBridge rule target configuration
- Confirm IAM role has `states:StartExecution` permission

### Data Quality Failures
- Review bad data in S3 quarantine location
- Adjust quality rules if needed
- Check CloudWatch logs for detailed error messages

### ETL Job Failures
- Verify Redshift connectivity
- Check Glue job logs in CloudWatch
- Ensure proper IAM permissions for Redshift access

---

## 📚 Additional Resources

- [AWS Glue Data Quality Documentation](https://docs.aws.amazon.com/glue/latest/dg/glue-data-quality.html)
- [Step Functions Best Practices](https://docs.aws.amazon.com/step-functions/latest/dg/best-practices.html)
- [EventBridge Event Patterns](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns.html)

---

## 👤 Author

**Your Name**
- Email: prajalkv620@gmail.com
- LinkedIn: https://www.linkedin.com/in/prajwal-kv-data-ai-engineer/
- 

---


---

## 🙏 Acknowledgments

- AWS Documentation and Best Practices
- AWS Community for troubleshooting support
- Open-source data engineering community

---

**⭐ If you find this project useful, please star it on GitHub!**
