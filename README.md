# Customer Support Chatbot with Amazon Bedrock

## 1. Project Overview

This project implements an intelligent customer support chatbot using
**Amazon Bedrock Flows**, **AWS Lambda**, **Amazon DynamoDB**, **Amazon S3**,
and **Amazon Bedrock Evaluations**.

The chatbot automatically classifies incoming customer requests and routes
them to the appropriate response path.

The implemented paths are:

- **Bug Reports** — identifies software issues, collects the required bug
  information, invokes a Lambda function, and integrates with DynamoDB for
  bug-report persistence.
- **Platform Questions** — answers questions covered by the embedded platform FAQ.
- **Uncovered Platform Questions** — informs the customer when the requested
  information is not available in the FAQ and directs them to customer support.
- **Other Requests** — handles requests outside the supported bug-report and FAQ
  categories and directs the customer to appropriate support.

The project also includes an automated testing and evaluation pipeline using:

- `flow-tests.json`
- `generate-eval-dataset.py`
- Amazon S3
- Amazon Bedrock Evaluations
- JSONL evaluation datasets

---

## 2. Architecture

```text
                         Customer Query
                              |
                              v
                       Flow Input Node
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
                 Bug Prompt FAQ Prompt Other Prompt
                      |
                      v
                Lambda Function
                      |
                      v
                   DynamoDB
                  BugReports
```

### Core Components

- **Amazon Bedrock Flow**  
  Receives customer messages, classifies them, and routes them using a Condition node.

- **Classifier Prompt**  
  Classifies each incoming message into the appropriate routing category.

- **Condition Node**  
  Evaluates the classifier output and sends the request to the corresponding path.

- **Bug Report Path**  
  Handles software problems and collects the information required to create a useful bug report.

- **AWS Lambda**  
  Processes the bug-report information and creates the bug-report record.

- **Amazon DynamoDB**  
  Stores bug-report records in the `BugReports` table.

- **Platform FAQ Prompt**  
  Uses the embedded FAQ content to answer supported platform questions.

- **Other Request Path**  
  Handles requests outside the supported bug and FAQ categories.

- **Amazon S3**  
  Stores the generated evaluation dataset and evaluation-related artifacts.

- **Amazon Bedrock Evaluations**  
  Evaluates the chatbot responses against the generated evaluation dataset.

---

## 3. Classification and Routing

The classifier processes every incoming customer message and determines which path should handle the request.

### Bug Reports

The bug-report path handles software problems and functional issues.

Examples include:

- Checkout button not working
- Application errors
- Functional problems
- Software bugs

These requests are routed to the bug-report path.

### Platform Questions

Questions related to documented platform functionality are routed to the FAQ prompt.

For example:

> How do I reset my password?

The chatbot responds using the embedded FAQ information.

### Other Requests

Requests outside the supported bug-report and FAQ categories are routed to the other-request response.

For example:

> I would like to discuss a sponsorship partnership.

The chatbot responds by directing the customer toward appropriate support.

---

## 4. Bug Report Path

The bug-report path is designed to collect the information required to create a useful bug report.

The required information includes:

- Bug description
- Steps to reproduce
- Environment information

Environment information can include details such as:

- Browser
- Operating system
- Device

### Bug Report Flow

The general flow is:

```text
Customer Bug Report
        |
        v
Classifier
        |
        v
Condition Node
        |
        v
Bug Path
        |
        v
Bug Information Collection
        |
        v
Lambda Function
        |
        v
DynamoDB BugReports Table
```

During testing, the flow successfully executed the Lambda function and returned a generated ticket ID and `OPEN` status.

Example response:

```text
Thank you for your bug report. Your ticket has been successfully created.

Ticket ID: <generated-ticket-id>
Status: OPEN
```

The Bedrock Flow execution trace also showed the Lambda function node completing successfully.

---

## 5. Platform FAQ Path

The FAQ path uses an embedded platform FAQ to answer supported customer questions.

The prompt is designed to keep responses within the information available in the FAQ.

### Covered Question

Example:

> How do I reset my password?

The flow returned an FAQ-based response explaining that the customer can use the **Forgot password** link on the sign-in page.

### Uncovered Question

Example:

> Can I change the color of a product after ordering?

When the requested information was not available in the FAQ, the flow directed the customer to contact support instead of inventing an answer.

This provides a separate path for questions that are outside the available FAQ information.

---

## 6. Other Request Path

Requests that do not belong to the supported bug-report or FAQ categories are routed to the Other Request path.

Example:

> I would like to discuss a sponsorship partnership.

The flow identifies the request as outside the normal support categories and returns a response directing the customer toward appropriate support.

This prevents unnecessary processing for requests that cannot be handled by the FAQ or bug-report workflow.

---

## 7. Testing

The project uses `flow-tests.json` to define test scenarios for the major conversation paths.

The test scenarios include:

1. Bug report
2. Covered platform FAQ question
3. Uncovered platform FAQ question
4. Other request

### Example Test Cases

#### Bug Report

```text
The checkout button is not working. To reproduce it, I add an item
to my cart, go to checkout, and click the checkout button, but nothing
happens. Environment: Chrome browser on Windows 11 desktop.
```

The flow classified the request as a bug and successfully executed the bug-report path.

#### Platform FAQ

```text
How do I reset my password?
```

The flow returned an answer based on the embedded FAQ.

#### Uncovered FAQ Question

```text
Can I change the color of a product after ordering?
```

The flow determined that the information was not available in the FAQ and directed the customer to support.

#### Other Request

```text
I would like to discuss a sponsorship partnership.
```

The flow routed the request through the Other Request path.

---

## 8. Evaluation Dataset Generation

The project includes `generate-eval-dataset.py` for automatically executing the test cases against the deployed Bedrock Flow and generating an evaluation dataset.

The script:

1. Reads test scenarios from `flow-tests.json`.
2. Sends the test inputs to the deployed Bedrock Flow.
3. Collects the generated responses.
4. Creates a JSONL evaluation dataset.
5. Uploads the dataset to Amazon S3.
6. Supports the subsequent Bedrock Evaluation workflow.

Example command:

```bash
python generate-eval-dataset.py \
  --tests-json flow-tests.json \
  --flow-id <FLOW_ID> \
  --flow-alias-id <FLOW_ALIAS_ID> \
  --region us-east-1 \
  --out-jsonl eval-dataset.jsonl
```

The actual Flow ID and Alias ID used for the final deployment are retained in the project configuration and evaluation artifacts.

---

## 9. Amazon S3

The generated evaluation dataset is uploaded to an Amazon S3 bucket for use by the Bedrock Evaluation workflow.

The project uses S3 to store evaluation-related data rather than keeping the evaluation dataset only on the local environment.

The generated JSONL dataset contains the test prompts, reference responses, and model responses used for evaluation.

---

## 10. Amazon Bedrock Evaluation

An Amazon Bedrock Evaluation Job was created using the generated evaluation dataset.

The evaluation uses an automated **LLM-as-a-judge** approach to compare the chatbot's responses against the expected reference responses.

The evaluation assessed response quality using metrics including:

- **Correctness**
- **Relevance**
- **Completeness**
- **Following Instructions**

The evaluation artifacts generated during the project are included in the repository.

---

## 11. Evaluation Results

The final evaluation results are retained in:

```text
evaluation-results-final.jsonl
```

The evaluation covered the major test scenarios:

- Bug report
- Covered FAQ question
- Uncovered FAQ question
- Other request

The final evaluation results showed strong performance across the tested scenarios.

The results should be interpreted together with the individual flow execution traces and functional tests rather than being treated as a replacement for functional testing.

---

## 12. Evidence

Screenshots demonstrating the implementation and testing are available in the
`screenshots/` directory.

The evidence includes:

- Complete flow classification and routing
- Bug-report execution trace
- Bug-report Lambda execution trace
- Platform FAQ response
- Uncovered FAQ response
- Other-request routing

The screenshots provide visual evidence of the implemented flow and its tested
behavior across the main request paths.

---

## 13. Project Files

```text
.
├── README.md
├── .gitignore
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
```

---

## 14. Technologies Used

- **Amazon Bedrock Flows**
- **Amazon Bedrock Evaluations**
- **AWS Lambda**
- **Amazon DynamoDB**
- **Amazon S3**
- **Python**
- **AWS CLI**
- **JSON**
- **JSONL**

---

## 15. Key Takeaways

This project demonstrates an end-to-end customer-support workflow using Amazon Bedrock Flows.

The implementation demonstrates:

- Prompt-based request classification
- Conditional routing
- Bug-report information collection
- Lambda-based backend processing
- DynamoDB integration for bug-report persistence
- FAQ-based response generation
- Handling of unsupported requests
- Automated flow testing
- Evaluation dataset generation
- Amazon S3 integration
- Amazon Bedrock model evaluation
The project combines **LLM-based routing with traditional AWS services** to create a structured customer-support workflow with testable and observable execution paths.
