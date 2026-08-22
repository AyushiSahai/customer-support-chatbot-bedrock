# Customer Support Chatbot with Amazon Bedrock

## 1. Project Overview

This project implements an intelligent customer support chatbot using
**Amazon Bedrock Flows**, **AWS Lambda**, and **Amazon DynamoDB**.

The chatbot automatically classifies incoming customer requests into
different categories and routes each request to the appropriate response path.

The implemented paths are:

- **Bug Reports** — collects the required bug information and creates a bug-report ticket.
- **Platform Questions** — answers questions covered by the embedded platform FAQ.
- **Uncovered Platform Questions** — informs the customer when the requested
  information is not available in the FAQ and directs them to support.
- **Other Requests** — routes requests such as sponsorship or partnership
  inquiries to human support.

The project also includes an automated testing and evaluation pipeline using
`flow-tests.json`, `generate-eval-dataset.py`, Amazon S3, and Amazon Bedrock
Evaluations.

---

## 2. Architecture

```text
                         Customer Query
                              |
                              v
                       FlowInputNode
                              |
                              v
                     Classifier Prompt
                              |
                              v
                       Condition Node
                       /      |       \
                     BUG     FAQ     OTHER
                      |       |        |
                      v       v        v
                  Bug Path  FAQ Path  Other Request
                      |       |        |
                      v       v        v
                    Prompt  FAQ Prompt Output
                      |
                      v
               Lambda Function
                      |
                      v
Core Components
Amazon Bedrock Flow
Receives customer messages.
Classifies requests.
Routes messages using a Condition node.
Classifier Prompt
Produces one of the supported categories:
BUG, PLATFORM_QUESTION, or OTHER.
Bug Report Path
Collects bug description, reproduction steps, and environment.
Invokes the configured Lambda function.
Creates a bug-report record in DynamoDB.
Platform FAQ Path
Uses the embedded FAQ content.
Answers questions covered by the FAQ.
Directs customers to support when the requested information is not covered.
Other Request Path
Routes requests that do not belong to the bug or FAQ categories
to a human-support response.
Amazon Bedrock Evaluation
Evaluates the generated responses against the test dataset.
3. Classification and Routing

The classifier processes every incoming customer message and determines
which path should handle the request.

Bug Reports

Examples include:

Checkout button not working
Application errors
Functional problems
Software bugs

These requests are routed to the bug-report path.

Platform Questions

Questions related to documented platform functionality are routed to the
FAQ prompt.

For example:

How do I reset my password?

The chatbot responds using the embedded FAQ information.

Other Requests

Requests outside the supported bug and FAQ categories are routed to the
human-support response.

For example:

I would like to discuss a sponsorship partnership.

4. Bug Report Path

The bug-report path collects the information required to create a useful
bug report:

Bug description
Steps to reproduce
Environment

A Lambda function is then invoked to create the bug report record in
DynamoDB.

During testing, the flow successfully executed the Lambda function and
returned a ticket ID and OPEN status.

Example response:

Thank you for your bug report. We have successfully created your ticket.

Ticket ID: <generated-ticket-id>
Status: OPEN

The execution trace confirms that the
LambdaFunctionNode_1 completed successfully.

5. Platform FAQ Path

The FAQ path uses the embedded platform FAQ to answer supported questions.

Covered Question

Example:

How do I reset my password?

The flow returned the FAQ-based answer explaining that the customer can
use the Forgot password link on the sign-in page.

Uncovered Question

Example:

Can I change the color of a product after ordering?

The flow correctly identified that the information was not available in the
FAQ and directed the customer to contact support.

6. Other Request Path

Requests outside the supported bug-report and FAQ categories are routed to
a human-support response.

Example:

I would like to discuss a sponsorship partnership.

The flow routes this request through the OtherRequest prompt and returns
a response directing the customer to the appropriate support channel.

7. Testing

The project uses flow-tests.json to define test scenarios for the major
conversation paths.

The final test dataset contains four scenarios:

Bug report
Platform FAQ question
FAQ-uncovered question
Other request

The test suite was executed using:

python generate-eval-dataset.py \
  --tests-json flow-tests.json \
  --flow-id HQM4BOM7JA \
  --flow-alias-id TSTALIASID \
  --region us-east-1 \
  --out-jsonl eval-dataset.jsonl

All four flow calls completed successfully and produced the evaluation
dataset.

8. Amazon S3 and Bedrock Evaluation

The generated evaluation dataset was uploaded to Amazon S3:

s3://udacity-agentic-engineer-c1-eval-158489067310/eval-dataset.jsonl

An Amazon Bedrock Evaluation Job was then created using the generated
dataset.

The evaluation job completed successfully.

The evaluation output was stored under the configured S3 evaluation-results
location and was also retained in:

evaluation-results-final.jsonl
9. Evaluation Results

The final evaluation results are included in:

evaluation-results-final.jsonl

The evaluation covered the four test scenarios and used automated
LLM-as-a-judge evaluation.

The results should be interpreted together with the individual test
responses rather than as a replacement for functional testing.

10. Evidence

Screenshots demonstrating the implementation and testing are available in
the screenshots/ directory.

The evidence includes:

Complete Bedrock Flow diagram
Bug-report execution
Bug-report Lambda trace
Platform FAQ response
FAQ-uncovered response
Other-request routing
Flow execution traces
11. Project Files
.
├── README.md
├── cloudformation-testing.yaml
├── cloudformation-tool.yaml
├── create_bug_report.py
├── eval-dataset.jsonl
├── evaluation-job-final.json
├── evaluation-job.json
├── evaluation-results-final.jsonl
├── evaluation-results.jsonl
├── flow-tests-template.json
├── flow-tests.json
├── generate-eval-dataset.py
├── online_shop_faq.md
├── requirements.txt
└── screenshots/
    ├── 01-Flow-diagram.png
    ├── 02-Bug-report-test.png
    ├── 03-Bug-report-trace.png
    ├── 04-Platform-FAQ-question.png
    ├── 05-Platform-FAQ-uncovered.png
    └── 06-Other-request.png
12. Technologies Used
Amazon Bedrock Flows
Amazon Bedrock Evaluations
AWS Lambda
Amazon DynamoDB
Amazon S3
Python
AWS CLI
JSON / JSONL
                 DynamoDB
                BugReports
