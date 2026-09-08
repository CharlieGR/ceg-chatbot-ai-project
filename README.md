# Customer Support Chatbot — Amazon Bedrock AgentCore

A customer support chatbot for a fictional online shop, built on the Amazon Bedrock AgentCore managed harness. A single system prompt (`project/starter/system_prompt.txt`) routes every customer message to exactly one of three behaviors — no classifier nodes, no condition graph:

- **Bug reports** — collects description, steps to reproduce, and environment over the conversation, then files a ticket via the `create_bug_report` tool (a Lambda function exposed through an AgentCore Gateway) and relays the ticket ID.
- **Platform questions** — answered from an embedded FAQ (`project/starter/online_shop_faq.md`).
- **Anything else** — a polite hand-off to the human support line (`1-800-555-0199`).

Model: `us.amazon.nova-pro-v1:0`, pinned everywhere. Region: `us-east-1`.

## Repo layout

- `project/starter/` — all working code: the system prompt, CloudFormation templates, the Lambda tool, setup/harness/chat scripts, the test suite, and the eval dataset generator
- `evidence/` — chat transcripts and console screenshots demonstrating each route working end to end
- `EVALUATION_OBSERVATIONS.md` — written observations on the Bedrock Evaluations results, referencing actual per-record scores

## What's in `evidence/`

- `chat_transcripts/01_bug_report.txt` — a full multi-turn bug-report conversation: follow-up questions, the `[tool call] bugreports___create_bug_report` line, and the ticket ID
- `chat_transcripts/02_faq_covered.txt`, `03_faq_uncovered.txt`, `04_out_of_scope.txt` — the other two routes and the FAQ hand-off case
- `bug-report-dynamodb-table-evidence.png` — the `bug-report-tool-stack-bug-reports` DynamoDB table with chatbot-created tickets, including the one from `01_bug_report.txt`
- `bedrock-eval-1.png`, `bedrock-eval-2.png` — the completed Bedrock Evaluations results page, all 9 test cases

## Testing and evaluation

`project/starter/harness-tests.json` covers all three routes plus edge cases (ambiguous, very short, prompt injection). `generate-eval-dataset.py` ran all 9 against the harness with zero `[HARNESS_ERROR]` entries, producing `project/starter/output_eval_dataset.jsonl`. A Bedrock Evaluations job (`support-chatbot-eval-run-1`, `Builtin.Correctness`, judge model `amazon.nova-pro-v1:0`) scored every record **1.0**.

See **[EVALUATION_OBSERVATIONS.md](EVALUATION_OBSERVATIONS.md)** for the full per-record breakdown, what the judge model's explanations actually said, and concrete follow-ups — including two real bugs found and fixed during manual testing that a single-turn eval suite structurally can't catch on its own (a cross-session memory leak in the harness config, and a prompt change that suppressed visible reasoning but broke tool-result accuracy).

## Setup

Deployment steps (CloudFormation stacks, Gateway registration, harness creation) follow `.project_steps/2.environment_setup.md` and `.project_steps/3.instructions.md`. `agentcore_config.json` in `project/starter/` records the resulting harness and gateway ARNs.
