# Customer Support Chatbot with Amazon Bedrock

## Project Overview

This project implements a customer support chatbot using Amazon Bedrock Flows. The chatbot classifies incoming customer messages and routes them to the appropriate response path.

### Supported Request Types

- **Bug reports** — collects bug details and creates a support ticket.
- **Platform questions** — answers questions covered by the platform FAQ.
- **Uncovered platform questions** — informs the customer when the FAQ does not contain the requested information and directs them to support.
- **Other requests** — routes requests requiring human assistance to the support team.

## Architecture

The flow uses:

- Amazon Bedrock Flows for classification and routing
- Amazon Bedrock prompts for response generation
- AWS Lambda for bug-report processing
- Amazon DynamoDB for storing bug-report tickets
- Amazon S3 and Amazon Bedrock Evaluations for evaluation

## Bug Report Path

For bug reports, the chatbot collects:

1. Bug description
2. Steps to reproduce
3. Environment

The collected information is passed to the Lambda function, which creates a bug-report ticket in DynamoDB and returns the ticket ID and status.

## Testing

The flow was tested with four scenarios:

1. Bug report
2. Platform FAQ question
3. Platform question not covered by the FAQ
4. Other customer-support request

The automated evaluation was completed using Amazon Bedrock Evaluations.

## Evidence

Screenshots demonstrating the implementation and test results are available in the `screenshots/` directory.

- Flow architecture and routing
- Bug-report execution and Lambda trace
- Platform FAQ response
- FAQ-uncovered response
- Other-request routing
- Test execution results

## Evaluation

The evaluation dataset and evaluation results are included in the repository:

- `flow-tests.json`
- `eval-dataset.jsonl`
- `evaluation-results-final.jsonl`
