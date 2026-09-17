# Telegram AI Agent with Amazon Search

## Overview

This n8n workflow implements a Telegram-based AI assistant that receives
user messages, processes them through an AI Agent, and sends the
generated response back to the user through Telegram.

The AI Agent is connected to:

-   An OpenAI Chat Model
-   Simple Memory for conversational context
-   Amazon product search through SerpApi

This workflow is designed to provide a conversational interface where
users can ask questions, request product searches, and receive responses
directly in Telegram.

------------------------------------------------------------------------

## Workflow Architecture

``` text
Telegram Trigger
       |
       v
    AI Agent
       |
       v
Send a text message

AI Agent connections:
- OpenAI Chat Model
- Simple Memory
- Amazon Search in SerpApi
```

------------------------------------------------------------------------

## Workflow Components

### 1. Telegram Trigger

**Node:** `Telegram Trigger`

The Telegram Trigger receives incoming messages from users and starts
the workflow.

**Configuration:**

-   Event type: Message
-   Telegram bot credentials
-   Incoming user message payload

**Possible use cases:**

-   Asking general questions
-   Requesting product information
-   Searching for Amazon products
-   Continuing an existing conversation

------------------------------------------------------------------------

### 2. AI Agent

**Node:** `AI Agent`

The AI Agent is the central processing and orchestration component of
the workflow.

It is responsible for:

1.  Receiving the user's Telegram message.
2.  Understanding the user's intent.
3.  Generating a response using the connected OpenAI Chat Model.
4.  Maintaining conversational context through memory.
5.  Calling the Amazon search tool when required.
6.  Returning a final response to the Telegram node.

The AI Agent can determine whether a request can be answered directly or
whether an external tool is needed.

------------------------------------------------------------------------

### 3. OpenAI Chat Model

**Node:** `OpenAI Chat Model`

The OpenAI Chat Model provides the language model used by the AI Agent.

It supports:

-   Natural-language understanding
-   Intent identification
-   Response generation
-   Tool-use reasoning
-   Summarization of search results
-   Conversational interactions

The selected model, credentials, system instructions, and model
parameters should be configured within n8n.

------------------------------------------------------------------------

### 4. Simple Memory

**Node:** `Simple Memory`

Simple Memory stores relevant conversation context for the AI Agent.

It can help the assistant:

-   Understand follow-up questions
-   Remember recent conversation details
-   Maintain continuity across interactions
-   Produce more context-aware responses

> The persistence, session identification, and retention behavior depend
> on the memory configuration in n8n.

For multi-user deployments, ensure that memory sessions are separated by
Telegram user or chat ID to prevent conversation data from being mixed.

------------------------------------------------------------------------

### 5. Amazon Search in SerpApi

**Node:** `Amazon search in SerpApi`

This tool enables the AI Agent to search for Amazon-related product
information through SerpApi.

**Potential use cases:**

-   Searching for products
-   Finding product listings
-   Comparing product information
-   Reviewing product titles and prices returned by search
-   Supporting shopping-related questions

**Configuration requirements:**

-   SerpApi API credentials
-   Amazon search engine or query configuration
-   Appropriate country and language settings
-   Result parsing and response formatting

> Search results should be presented as retrieved information. Product
> availability, pricing, delivery estimates, and stock status should be
> verified before being treated as definitive because they can change.

------------------------------------------------------------------------

### 6. Send a Text Message

**Node:** `Send a text message`

This Telegram node sends the AI Agent's final response back to the user.

The response may include:

-   A direct answer
-   A summary of Amazon search results
-   Product recommendations based on search criteria
-   A clarification question
-   A message explaining that a tool or service is unavailable

The Telegram node should map the appropriate chat ID and response text
from the workflow execution data.

------------------------------------------------------------------------

## End-to-End Execution

1.  A user sends a message to the Telegram bot.
2.  The Telegram Trigger receives the message.
3.  The message is passed to the AI Agent.
4.  The AI Agent processes the request using the OpenAI Chat Model.
5.  The AI Agent checks Simple Memory when conversation context is
    needed.
6.  If the request involves Amazon products, the AI Agent invokes the
    SerpApi search tool.
7.  The tool returns search results to the AI Agent.
8.  The AI Agent generates a final response.
9.  The `Send a text message` node sends the response to the Telegram
    user.

------------------------------------------------------------------------

## Example Requests

  -----------------------------------------------------------------------
  Example request                     Expected behavior
  ----------------------------------- -----------------------------------
  "Explain workflow automation."      The AI Agent generates a direct
                                      answer.

  "Search Amazon for wireless         The Amazon SerpApi tool is invoked.
  headphones."                        

  "Compare the products you found."   The agent uses available search
                                      context to summarize results.

  "What did I ask earlier?"           Simple Memory may provide relevant
                                      conversation context.

  "Find a product under a specific    The agent searches using the
  budget."                            provided criteria when supported by
                                      the tool configuration.
  -----------------------------------------------------------------------

Actual behavior depends on the AI Agent prompt, model configuration,
memory settings, and tool permissions.

------------------------------------------------------------------------

## Required Integrations

  Integration        Purpose
  ------------------ ------------------------------------------
  Telegram Bot API   Receive user messages and send responses
  OpenAI             Provide the AI Agent's language model
  SerpApi            Search Amazon product information
  n8n                Workflow orchestration and execution

------------------------------------------------------------------------

## Configuration Checklist

-   [ ] Create and configure a Telegram bot.
-   [ ] Add Telegram credentials to n8n.
-   [ ] Configure the Telegram Trigger.
-   [ ] Connect the OpenAI Chat Model to the AI Agent.
-   [ ] Configure the AI Agent system prompt.
-   [ ] Configure Simple Memory and session handling.
-   [ ] Add SerpApi credentials.
-   [ ] Configure Amazon search parameters.
-   [ ] Connect the AI Agent output to the Telegram response node.
-   [ ] Map the correct Telegram chat ID.
-   [ ] Test a standard text request.
-   [ ] Test an Amazon product search request.
-   [ ] Test a follow-up question using memory.
-   [ ] Configure error handling and logging.

------------------------------------------------------------------------

## Security Considerations

-   Store API keys and bot credentials using n8n's credential manager.
-   Do not hardcode secrets in node fields, scripts, or exported
    workflow files.
-   Use separate memory sessions for different Telegram users.
-   Avoid exposing confidential user information in responses.
-   Validate external search results before presenting them as verified
    facts.
-   Apply rate limits where appropriate.
-   Monitor API usage and unexpected tool calls.
-   Review the AI Agent's permissions before production deployment.

------------------------------------------------------------------------

## Error Handling

The workflow should handle common failure scenarios, including:

-   Telegram trigger failures
-   OpenAI API errors
-   Memory session errors
-   SerpApi authentication failures
-   No Amazon search results
-   API timeouts
-   Invalid or incomplete user requests
-   Telegram message delivery failures

A recommended error-handling approach is:

1.  Capture the technical error securely.
2.  Log the relevant execution details.
3.  Return a clear and user-friendly message.
4.  Avoid exposing internal credentials or stack traces.
5.  Retry only when the failure is temporary and safe to retry.

------------------------------------------------------------------------

## Testing Plan

  -----------------------------------------------------------------------
  Test scenario                       Expected result
  ----------------------------------- -----------------------------------
  Basic text message                  The AI Agent returns a relevant
                                      response.

  Amazon product search               SerpApi is called and results are
                                      summarized.

  No search results                   The agent informs the user clearly.

  Follow-up question                  Memory provides relevant prior
                                      context.

  Ambiguous request                   The agent asks for clarification.

  Invalid API credentials             A controlled error is returned and
                                      logged.

  Telegram delivery failure           The workflow handles the failure
                                      appropriately.

  Multiple users                      User conversations remain isolated.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## Suggested Enhancements

-   Add a persistent database for long-term conversation history.
-   Add Telegram user authentication or allowlists.
-   Add rate limiting and usage monitoring.
-   Add structured logging and workflow execution tracking.
-   Add product filtering by price, rating, category, or availability.
-   Add explicit citations or source links for search results.
-   Add confirmation before performing sensitive or paid actions.
-   Add retry logic with exponential backoff.
-   Add multilingual support.
-   Add monitoring for response latency, tool failures, and API cost.
-   Add automated regression tests for the main workflow paths.

------------------------------------------------------------------------

## Operational Metrics

Recommended metrics include:

-   Number of Telegram messages received
-   AI Agent response latency
-   OpenAI request success and failure rate
-   Amazon search invocation count
-   SerpApi success and failure rate
-   Number of searches returning no results
-   Telegram response delivery success rate
-   Average conversation length
-   Memory retrieval failures
-   API usage and cost
-   User feedback and repeated failure patterns

------------------------------------------------------------------------

## Summary

This n8n workflow provides a lightweight Telegram AI assistant powered
by an OpenAI Chat Model. It combines conversational memory with Amazon
product search through SerpApi and delivers responses through Telegram.

The workflow demonstrates the integration of:

-   Messaging automation
-   AI Agent orchestration
-   Large language models
-   Conversation memory
-   External search tools
-   Telegram response delivery
