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
                 DynamoDB
                BugReports
