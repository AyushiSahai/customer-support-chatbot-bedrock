# Customer Support Chatbot — Rebuild & Interview Notes

## 1. Project Goal

Built a customer-support chatbot using Amazon Bedrock Flows.

The flow:
Customer Query → Classifier Prompt → Condition Node → BUG / FAQ / OTHER

BUG:
→ Bug information collection
→ Lambda
→ DynamoDB BugReports table
→ Ticket ID + OPEN status

FAQ:
→ Embedded FAQ Prompt
→ Answers supported questions
→ Escalates unsupported questions

OTHER:
→ Other Request response
→ Directs customer to appropriate support.

---

## 2. AWS Services Used

- Amazon Bedrock Flows
- Amazon Bedrock Evaluation
- AWS Lambda
- Amazon DynamoDB
- Amazon S3
- AWS IAM
- AWS CLI
- Python

---

## 3. Flow Architecture

Customer Query
    ↓
Flow Input Node
    ↓
Classifier Prompt
    ↓
Condition Node
    ├── BUG → Bug Path → Lambda → DynamoDB
    ├── FAQ → FAQ Prompt
    └── OTHER → Other Request Prompt

---

## 4. Classification

BUG:
Software problems and functional issues.

Examples:
- Checkout button not working
- Application errors
- Software bugs

PLATFORM_QUESTION:
Questions covered by the embedded FAQ.

Example:
- How do I reset my password?

OTHER:
Requests outside the supported bug and FAQ categories.

Example:
- Sponsorship/partnership request

---

## 5. Bug Report Workflow

Required information:

1. Bug description
2. Steps to reproduce
3. Environment

Environment can include:
- Browser
- Operating system
- Device

After collecting the information:

Flow
→ Bug path
→ Lambda
→ DynamoDB
→ ticketId
→ OPEN status

Important evidence:
A chatbot-generated ticket ID was queried in DynamoDB and the
matching record was found with status OPEN.

---

## 6. DynamoDB

Table:
BugReports-3fc045d0

Important attributes observed:

- ticketId
- createdAt
- description
- environment
- status
- stepsToReproduce

---

## 7. FAQ

The FAQ uses embedded FAQ content.

Covered questions:
Answered using the embedded FAQ.

Uncovered questions:
The chatbot does not invent an answer and directs the customer to support.

---

## 8. Testing

Test scenarios:

1. Bug report
2. Covered FAQ
3. Uncovered FAQ
4. Other request

Files:

flow-tests.json
generate-eval-dataset.py

The script executes the Flow and produces a JSONL evaluation dataset.

---

## 9. Evaluation

Evaluation approach:

Amazon Bedrock Evaluation
→ LLM-as-a-judge
→ Amazon Nova Pro

BYOI source:
my-flow-app

Original evaluation:
Builtin.Helpfulness = 0.75

Later evaluation:
Builtin.Helpfulness = 0.79
Builtin.Correctness = 1.00
Builtin.Harmfulness = 0.00

---

## 10. S3

Used S3 for:

- Evaluation dataset
- Evaluation output/results

Region:
us-east-1

---

## 11. Important AWS Console Locations

Bedrock:
Amazon Bedrock → Flows

Evaluations:
Amazon Bedrock → Evaluations

DynamoDB:
DynamoDB → Tables → BugReports-3fc045d0 → Explore items

Lambda:
AWS Lambda → Functions

S3:
Amazon S3 → evaluation bucket

---

## 12. CLI Commands Used

Check AWS credentials:

aws sts get-caller-identity

Check region:

aws configure get region

Generate evaluation dataset:

python generate-eval-dataset.py \
  --tests-json flow-tests.json \
  --flow-id <FLOW_ID> \
  --flow-alias-id <FLOW_ALIAS_ID> \
  --region us-east-1 \
  --out-jsonl eval-dataset.jsonl

Upload files to S3:

aws s3 cp <file> s3://<bucket>/<path> --region us-east-1

List S3:

aws s3 ls s3://<bucket>/<path>/

---

## 13. Important Project IDs / Configuration

Region:
us-east-1

Flow ID:
HQM4B0M7JA

Flow Alias ID:
None currently configured

Test execution alias:
TSTALIASID

S3 evaluation bucket:
udacity-agentic-engineer-c1-eval-158489067310

S3 dataset path:
s3://udacity-agentic-engineer-c1-eval-158489067310/eval-dataset.jsonl

Evaluation results path:
s3://udacity-agentic-engineer-c1-eval-158489067310/evaluation-results/

DynamoDB table:
BugReports-3fc045d0

Lambda function:
create-bug-report-3fc045d0

Evaluation source name:
my-flow-app

Evaluator:
Amazon Nova Pro 1.0

---

## 14. Interview Explanation

### 30-second explanation

"I built a customer-support chatbot using Amazon Bedrock Flows. I used a
prompt-based classifier and a Condition node to route requests into three
paths: bug reports, platform FAQs, and other requests. For bug reports, the
flow collects the description, reproduction steps, and environment, then
invokes Lambda to create a ticket and persist it in DynamoDB. The FAQ path is
restricted to embedded documentation, while unsupported requests are
redirected to support. I also created automated flow tests, generated a
JSONL evaluation dataset, uploaded it to S3, and evaluated the responses
using Amazon Bedrock's LLM-as-a-judge evaluation."

### If asked "Why Bedrock Flows?"

"Bedrock Flows gave me a visual way to define the orchestration and routing
logic between prompt nodes, conditions, and backend AWS services."

### If asked "Why Lambda?"

"Lambda provides the backend integration point for processing the structured
bug report and persisting it in DynamoDB."

### If asked "Why DynamoDB?"

"The bug reports are structured records with attributes such as ticket ID,
description, reproduction steps, environment, and status, which makes
DynamoDB a suitable serverless persistence layer."

### If asked "How did you evaluate it?"

"I created four representative test scenarios, generated a JSONL evaluation
dataset from the Flow executions, uploaded it to S3, and used Amazon Bedrock
Evaluation with Amazon Nova Pro as the LLM judge. The final correctness
evaluation produced a score of 1.00."

### If asked "How did you verify the bug report?"

"I ran the bug-report path through the Flow, received a generated ticket ID
with OPEN status, and queried DynamoDB using that exact ticket ID. The
matching record was returned from the BugReports table."

---

## 15. Important Limitations

- FAQ content is embedded rather than dynamically retrieved from a Knowledge Base.
- High-volume workloads may require quota/concurrency planning.
- Multi-turn bug-report context can require additional state-management
  refinement.
- The project was built using the AWS environment provided by the course.

---

## 16. What I Would Improve

If I had more time, I would:

1. Move FAQ content to an Amazon Bedrock Knowledge Base.
2. Improve multi-turn conversation state.
3. Add stronger automated test coverage.
4. Add monitoring and logging.
5. Improve response helpfulness for bug-report and other-request scenarios.
6. Add more evaluation metrics and test cases.
