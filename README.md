# n8n Conversational Slack-to-LinkedIn Bot

This n8n workflow creates a stateful, conversational AI bot that lives in Slack. It can accept text or voice messages, transcribe them, understand your intent (draft vs. post), manage chat history in a database, write LinkedIn posts for you, and publish them after your approval.

## ✨ Features

* **Slack Interface:** Triggered by messages in a Slack channel from a specific user.
* **Multi-modal Input:** Accepts both typed text and `.m4a` voice notes.
* **Voice Transcription:** Uses OpenAI Whisper to transcribe audio messages.
* **Stateful Conversation:** Connects to a Postgres database to store and retrieve chat history, allowing for threaded, multi-turn conversations.
* **AI Intent Classification:** An LLM (`Text Classifier`) determines if your message is a new draft request or a final "send" command.
* **Drafting Workflow:** An AI agent (`Write LinkedIn Post`) drafts professional, emoji-free content based on your request and the conversation history.
* **Review & Approval:** The bot posts the draft back to the Slack thread for you to review.
* **Publishing Workflow:** When you say "post it," a separate agent (`Send LinkedIn Post`) extracts the final text.
* **Automated Posting:** The bot:
    1.  Publishes the post to LinkedIn.
    2.  Logs the post in a Google Sheet.
    3.  Posts a "first comment" (e.g., with a link) via an HTTP Request.
* **Confirmation:** The bot replies in the Slack thread to confirm the post is live.

## ⚙️ Workflow Breakdown

1.  **Input:** A `Slack Trigger` catches a new message (text or voice) from you.
2.  **Transcribe:** If the message is a voice file, it's downloaded and transcribed by the `OpenAI` (Whisper) node.
3.  **Memory:** The workflow pulls the conversation history from your `Postgres` database based on the Slack thread ID.
4.  **Classify:** A `Text Classifier` (OpenAI) analyzes the new message and history to decide if you want to "Draft" or "Send".
5.  **Draft Path:**
    * The `Write LinkedIn Post` agent drafts the content.
    * The `Send LinkedIn Draft to User` agent formats a reply.
    * A `Slack` node sends the draft back to the thread for review.
6.  **Send Path:**
    * The `Send LinkedIn Post` agent extracts the final post text from the chat history.
    * `Update LinkedIn Done` logs the post in a `Google Sheet`.
    * The `LinkedIn` node publishes the post.
    * An `HTTP Request` node posts the first comment.
    * `Send Confirmation` agent creates a success message, which is posted back to Slack.
7.  **Memory Update:** The bot's reply is saved back to the `Postgres` database to continue the conversation.

## 🚀 Setup Guide

1.  **Import:** Download `On_The_Go_LinkedIn_Posting_Bot.json` and import it into n8n.
2.  **Database Setup:** Set up a Postgres database with a table (e.g., `n8n_rodger_chat`) to store `session_id` (text) and `message` (jsonb).
3.  **Connect Credentials:**
    * **Slack:** You need two types:
        * `slackApi` (Bot Token) for the `Slack Trigger` and `Slack` message nodes.
        * `slackOAuth2Api` (User Token, with `files:read` scope) for the `HTTP Request1` node (to download voice files).
    * **OpenAI:** Add your `openAiApi` key for the `OpenAI` (transcription) and all `OpenAI Chat Model` (classification/drafting) nodes.
    * **Postgres:** Add your `postgres` credentials to all Postgres nodes.
    * **Google Sheets:** Add your `googleSheetsOAuth2Api` to the `Update LinkedIn Done` node.
    * **LinkedIn:** Add your `linkedInOAuth2Api` to the `LinkedIn` and `HTTP Request` (comment) nodes.
4.  **Configure Nodes:**
    * **Slack Trigger:** Set the `Channel ID` and `User ID(s)` to listen to.
    * **Postgres Nodes:** Update the `Table` name if you used a different one.
    * **Update LinkedIn Done (Sheets):** Set your `Document ID` and `Sheet Name`.
    * **LinkedIn Node:** Set your `Person` URN.
    * **HTTP Request (Comment Node):** Update the `URL` (with your post URN) and `actor` (with your person URN) in the `JSON Body`.
    * **All Slack Message Nodes:** Ensure the `Channel ID` is set correctly for replies.
5.  **Activate:** Save and activate the workflow.

## 📄 License

MIT
