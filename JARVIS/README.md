# JARVIS - Multi-Agent AI Assistant

JARVIS is an advanced n8n-based multi-agent AI system that serves as a personal assistant capable of managing emails, calendar events, creating content, searching the web, and performing calculations. It uses GPT-4o as the primary language model and orchestrates multiple specialized sub-agents to handle different tasks.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Components](#components)
3. [Setup Instructions](#setup-instructions)
4. [API Keys & Credentials Required](#api-keys--credentials-required)
5. [Agents & Their Capabilities](#agents--their-capabilities)
6. [System Prompts](#system-prompts)
7. [Usage Examples](#usage-examples)
8. [Workflow Structure](#workflow-structure)
9. [Troubleshooting](#troubleshooting)

---

## Architecture Overview

JARVIS is built on a hub-and-spoke architecture with a central orchestrator that routes user queries to specialized sub-agents:

```
User Request (Webhook)
    ↓
JARVIS (Main Agent)
    ↓
Routing Layer (Decides which agent to use)
    ↓
┌─────────────────────────────────────────────────────┐
│   Specialized Sub-Agents                            │
├─────────────────────────────────────────────────────┤
│ • Email Agent      - Gmail operations               │
│ • Calendar Agent   - Google Calendar management     │
│ • Content Creator  - Blog post generation           │
│ • Contact Agent    - Contact management             │
│ • Tavily Search    - Web search                     │
│ • Calculator       - Mathematical operations        │
└─────────────────────────────────────────────────────┘
```

The system uses **conversation memory** to maintain context across interactions and LangChain-based agents for intelligent decision-making.

---

## Components

### 1. **JARVIS.json** (Main Orchestrator)
The central hub that receives user queries and routes them to appropriate agents.

**Key Nodes:**
- **Webhook**: Entry point for all user requests
- **Jarvis Agent**: Main orchestrator with routing logic
- **OpenAI 4o Model**: GPT-4o for decision-making
- **Simple Memory**: Maintains conversation context
- **Sub-Agent Tools**: References to all specialized agents

### 2. **__Email_Agent.json**
Handles all Gmail-based email operations.

**Capabilities:**
- Send emails (HTML formatted, signed as "Nate")
- Create email drafts
- Get/retrieve emails with filters
- Reply to emails
- Label emails
- Mark emails as unread
- Get available labels

### 3. **__Calendar_Agent.json**
Manages Google Calendar events and scheduling.

**Capabilities:**
- Create events (solo or with attendees)
- Get/retrieve calendar events
- Update event details (time, duration)
- Delete events
- Default calendar: nateherk88@gmail.com

### 4. **__Content_Creator_Agent.json**
Generates blog posts and written content using Claude.

**Capabilities:**
- Create blog posts with research
- Uses Tavily for web research
- Generates HTML-formatted content
- Includes citations with hyperlinks
- Uses Anthropic Claude for generation

---

## Setup Instructions

### Prerequisites
- n8n instance (self-hosted or cloud)
- Gmail account with OAuth2 configured
- Google Calendar with OAuth2 configured
- OpenAI API key (GPT-4o access)
- Anthropic API key (Claude models)
- Tavily API key (web search)

### Step 1: Install n8n Workflows

1. Log in to your n8n instance
2. Create a new workflow
3. Go to **Workflows** → **Import from File**
4. Import the following in order:
   - `__Calendar_Agent.json`
   - `__Email_Agent.json`
   - `__Content_Creator_Agent.json`
   - `JARVIS.json` (main workflow)

### Step 2: Configure API Credentials

#### OpenAI Configuration
1. In the main JARVIS workflow, click on **"4o"** node
2. Create/select an OpenAI API credential:
   - API Key: `your-openai-api-key`
   - Model: `gpt-4o`
3. In sub-agents, do the same for each "OpenAI Chat Model" node

#### Gmail Configuration
1. For Email Agent, click on any Gmail node
2. Set up Gmail OAuth2:
   - Go to Google Cloud Console
   - Create OAuth2 credentials
   - Configure consent screen
   - Add the n8n callback URL
3. Authenticate your Gmail account

#### Google Calendar Configuration
1. For Calendar Agent, click on any Google Calendar node
2. Set up Google Calendar OAuth2:
   - Use the same Google Cloud credentials
   - Ensure Calendar API is enabled
3. Update the calendar email if needed (currently: `nateherk88@gmail.com`)

#### Tavily Search Configuration
1. In Content Creator Agent, click **"Tavily"** node
2. Add your Tavily API key to the JSON body:
   ```json
   {
     "api_key": "YOUR-TAVILY-API-KEY",
     "query": "{searchTerm}",
     "search_depth": "basic",
     "include_answer": true,
     "max_results": 3
   }
   ```

#### Anthropic Configuration
1. In Content Creator Agent, click **"Anthropic Chat Model"** node
2. Create/select an Anthropic API credential:
   - API Key: `your-anthropic-api-key`

### Step 3: Link Sub-Agents to Main Workflow

In the **JARVIS.json** main workflow:

1. **Email Agent Tool Node:**
   - Click the "Email Agent" node
   - Ensure it references `__Email_Agent.json`
   - Check workflow ID is correct

2. **Calendar Agent Tool Node:**
   - Click the "Calendar Agent" node
   - Ensure it references `__Calendar_Agent.json`
   - Check workflow ID is correct

3. **Content Creator Tool Node:**
   - Click the "Content Creator Agent" node
   - Ensure it references `__Content_Creator_Agent.json`
   - Check workflow ID is correct

4. **Contact Agent Tool Node:**
   - Map to your contact management workflow
   - Update the reference ID

### Step 4: Test the Setup

1. Activate all workflows:
   - Main JARVIS workflow
   - All sub-agent workflows

2. Get the webhook URL from JARVIS:
   - Click "Webhook" node
   - Copy the webhook URL

3. Test with a curl request:
   ```bash
   curl -X POST https://your-n8n-instance/webhook/n8n \
     -H "Content-Type: application/json" \
     -d '{"query": "what is 2+2?"}'
   ```

4. Expected response:
   ```json
   {
     "response": "2 + 2 = 4"
   }
   ```

---

## API Keys & Credentials Required

| Service | Credential | Purpose |
|---------|-----------|---------|
| OpenAI | API Key | GPT-4o model for main orchestration |
| Google | OAuth2 | Gmail and Google Calendar access |
| Anthropic | API Key | Claude model for content creation |
| Tavily | API Key | Web search for content research |

### Where to Get Keys

- **OpenAI**: https://platform.openai.com/api-keys
- **Google Cloud**: https://console.cloud.google.com
- **Anthropic**: https://console.anthropic.com
- **Tavily**: https://tavily.com/

---

## Agents & Their Capabilities

### 1. Email Agent

**System Prompt:**
```
You are an email management assistant. All emails must be formatted 
professionally in HTML and signed off as "Nate."

Tools:
- Send Email: Send composed emails
- Create Draft: Create email drafts
- Get Emails: Retrieve emails with optional filters
- Get Labels: Retrieve available Gmail labels
- Mark Unread: Mark emails as unread (requires message ID)
- Label Email: Add labels to emails (requires message ID and label ID)
- Email Reply: Reply to emails (requires message ID)

Notes:
- Get the message ID before marking, labeling, or replying
- Emails are formatted as HTML
- All emails are signed as "Nate"
- Current date/time: {{ $now }}
```

**Usage Examples:**
```
- "Send John an email asking about the meeting time"
- "Get my unread emails from the last week"
- "Draft a professional response to that email"
- "Mark the email from Sarah as unread"
- "Add the urgent label to emails from my boss"
```

---

### 2. Calendar Agent

**System Prompt:**
```
You are a calendar assistant managing events and scheduling.

Tools:
- Create Event with Attendee: For events with participants
- Create Event: For solo events
- Get Events: Fetch calendar schedules
- Delete Event: Remove events (requires event ID from Get Events)
- Update Event: Modify event details (requires event ID)

Requirements:
- Calendar: nateherk88@gmail.com
- If no duration specified, assume 1 hour
- Get event ID before updating or deleting

Current date/time: {{ $now }}
```

**Usage Examples:**
```
- "Schedule a meeting with John tomorrow at 2 PM"
- "What's on my calendar for Friday?"
- "Create a 30-minute call with Sarah at 3 PM"
- "Delete the meeting at 4 PM tomorrow"
- "Change the 2 PM meeting to 3 PM"
```

---

### 3. Content Creator Agent

**System Prompt:**
```
You are a skilled AI blog writer specializing in engaging, 
well-structured content.

Tools:
- Tavily: Search the web for research

Features:
- Creates SEO-optimized blog posts
- Formats content as HTML with proper tags
- Includes hyperlinked citations from sources
- Maintains natural, human-like tone
- Varied sentence structures
- Factually accurate and well-researched

Output Format:
- HTML with <h1>, <h2>, <p>, <ul><li>, <a href=""> tags
- Clickable hyperlinks to sources
- Proper heading hierarchy
```

**Usage Examples:**
```
- "Write a blog post about artificial intelligence"
- "Create content about the latest trends in web development"
- "Generate a comprehensive guide on machine learning"
```

---

### 4. Main JARVIS Agent

**System Prompt:**
```
You are the ultimate personal assistant. Your job is to route 
the user's query to the correct tool.

Available Tools:
- emailAgent: Email management and sending
- calendarAgent: Calendar and event management
- contactAgent: Contact information and management
- contentCreator: Blog post and content creation
- Tavily: Web search and research
- Calculator: Mathematical operations

Routing Rules:
1. For email/draft actions → emailAgent
2. For calendar/scheduling → calendarAgent
3. For contact lookup → contactAgent first (get email/info)
4. For blog/content → contentCreator
5. For web search → Tavily
6. For calculations → Calculator

Important:
- Some actions require lookups first (e.g., get email before sending)
- Contact agent must be called before email/calendar actions with unknown contacts
- Always pass contact information when sending to agents

Current date/time: {{ $now }}
```

---

## System Prompts

### JARVIS Main Orchestrator Prompt

```
# Overview
You are the ultimate personal assistant. Your job is to send the user's 
query to the correct tool. You should never be writing emails, or creating 
even summaries, you just need to call the correct tool.

## Tools
- emailAgent: Use this tool to take action in email
- calendarAgent: Use this tool to take action in calendar
- contactAgent: Use this tool to get, update, or add contacts
- contentCreator: Use this tool to create blog posts
- Tavily: Use this tool to search the web

## Rules
- Some actions require you to look up contact information first. For the 
  following actions, you must get contact information and send that to the 
  agent who needs it:
  - sending emails
  - drafting emails
  - creating calendar event with attendee

## Examples
1) Input: send an email to nate herkelman asking him what time he wants 
   to leave
   - Action: Use contactAgent to get nate herkelman's email
   - Action: Use emailAgent to send the email. You will pass the tool a 
     query like "send nate herkelman an email to ask what time he wants 
     to leave. here is his email: [email address]"
   - Output: The email has been sent to Nate Herkelman. Anything else 
     I can help you with?

## Final Reminders
Here is the current date/time: {{ $now }}
```

### Email Agent Prompt

```
# Overview
You are an email management assistant. All emails must be formatted 
professionally in HTML and signed off as "Nate."

**Email Management Tools**
- Use "Send Email" to send emails.
- Use "Create Draft" if the user asks for a draft.
- Use "Get Emails" to retrieve emails when requested.
- Use "Get Labels" to retrieve labels.
- Use "Mark Unread" to mark an email as unread. You must use "Get Emails" 
  first so you have the message ID of the email to flag.
- Use "Label Email" to flag an email. You must use "Get Emails" first so 
  you have the message ID of the email to flag. Then you must use 
  "Get Labels" so you have the label ID.
- Use "Email Reply" to reply to an email. You must use "Get Emails" first 
  so you have the message ID of the email to reply to.

## Final Notes
- Here is the current date/time: {{ $now }}
```

### Calendar Agent Prompt

```
# Overview
You are a calendar assistant. Your responsibilities include creating, 
getting, and deleting events in the user's calendar.

**Calendar Management Tools**
- Use "Create Event with Attendee" when an event includes a participant.
- Use "Create Event" for solo events.
- Use "Get Events" to fetch calendar schedules when requested.
- Use "Delete Event" to delete an event. You must use "Get Events" first 
  to get the ID of the event to delete.
- Use "Update Event" to update an event. You must use "Get Events" first 
  to get the ID of the event to update.

## Final Notes
Here is the current date/time: {{ $now }}
If a duration for an event isn't specified, assume it will be one hour.
```

### Content Creator Agent Prompt

```
# Overview
You are a skilled AI blog writer specializing in engaging, well-structured, 
and informative content. Your writing style is clear, compelling, and 
tailored to the target audience. You optimize for readability, SEO, and 
value, ensuring blogs are well-researched, original, and free of fluff.

## Tools
Tavily - Use this to search the web about the requested topic for the 
blog post.

## Blog Requirements
Format all blog content in HTML, using proper headings (<h1>, <h2>), 
paragraphs (<p>), bullet points (<ul><li>), and links (<a href="URL">) 
for citations. All citations from the Tavily tool must be preserved, with 
clickable hyperlinks so readers can access the original sources.

Maintain a natural, human-like tone, use varied sentence structures, and 
include relevant examples or data when needed. Structure content for easy 
reading with concise paragraphs and logical flow. Always ensure factual 
accuracy and align the tone with the intended brand or purpose.
```

---

## Usage Examples

### Example 1: Sending an Email
**User Request:**
```
"Send an email to john@example.com asking about the project deadline"
```

**Process:**
1. JARVIS receives the query
2. Routes to emailAgent
3. Email Agent generates HTML email
4. Gmail sends the email
5. Response: "Email sent to john@example.com"

### Example 2: Scheduling a Meeting
**User Request:**
```
"Schedule a 30-minute meeting with Sarah at 3 PM tomorrow"
```

**Process:**
1. JARVIS receives the query
2. Routes to contactAgent to get Sarah's email
3. Routes to calendarAgent with Sarah's email
4. Calendar creates event with attendee
5. Response: "Meeting scheduled with Sarah for tomorrow at 3 PM"

### Example 3: Creating a Blog Post
**User Request:**
```
"Write a blog post about machine learning trends"
```

**Process:**
1. JARVIS routes to contentCreator
2. Content Creator uses Tavily to search for latest ML trends
3. Claude generates SEO-optimized HTML blog post
4. Response: HTML blog post with citations

### Example 4: Web Search
**User Request:**
```
"What are the latest developments in AI?"
```

**Process:**
1. JARVIS routes to Tavily
2. Tavily searches the web
3. Returns top results with summaries
4. Response: Search results with links

---

## Workflow Structure

### Directory Structure
```
JARVIS/
├── JARVIS.json                 # Main orchestrator workflow
├── __Email_Agent.json          # Email management sub-agent
├── __Calendar_Agent.json       # Calendar management sub-agent
├── __Content_Creator_Agent.json # Blog/content creation agent
└── README.md                   # This file
```

### Workflow Connections

**JARVIS.json connections:**
```
Webhook → Jarvis (Main Agent) → Respond to Webhook
                ↓
         (Routes to sub-agents via AI tools)
                ↓
    Email Agent, Calendar Agent, Contact Agent, 
    Content Creator Agent, Tavily, Calculator
```

**Email Agent connections:**
```
When Executed → Email Agent → Success/Try Again
                    ↓
            (Uses Gmail tools)
                ↓
    Send Email, Create Draft, Get Emails, etc.
```

**Calendar Agent connections:**
```
When Executed → Calendar Agent → Success/Try Again
                    ↓
            (Uses Google Calendar tools)
                ↓
    Create Event, Get Events, Update Event, Delete Event
```

**Content Creator Agent connections:**
```
When Executed → Content Creator Agent → Response/Try Again
                        ↓
                (Uses Claude + Tavily)
                        ↓
                Blog Post Generation
```

---

## Troubleshooting

### Issue: Webhook returns "error" instead of response

**Solution:**
1. Check if all sub-agents are active
2. Verify API credentials are valid
3. Check n8n logs for detailed error messages
4. Ensure workflows are properly linked

### Issue: Email not being sent

**Solution:**
1. Verify Gmail OAuth2 is properly configured
2. Check email address format
3. Ensure "Send Email" node has correct credentials
4. Verify email HTML formatting (use HTML format)

### Issue: Calendar events not being created

**Solution:**
1. Verify Google Calendar OAuth2 is active
2. Update the calendar email address if different from `nateherk88@gmail.com`
3. Ensure attendee email is in correct format
4. Check Calendar API is enabled in Google Cloud

### Issue: Content Creator not generating blog posts

**Solution:**
1. Verify Anthropic API key is valid
2. Check Tavily API key in the HTTP tool
3. Ensure Tavily search is returning results
4. Check Claude model availability

### Issue: "Unable to perform task" response

**Solution:**
1. This indicates the agent encountered an error
2. Check the specific sub-agent workflow for errors
3. Verify all required tools are properly configured
4. Test individual agents independently

### Debugging Tips

1. **Test Individual Workflows:**
   - Activate each sub-agent individually
   - Use n8n's "Execute" button to test manually
   - Check execution logs for detailed errors

2. **Check Credentials:**
   - All credentials should show "✓" in the credentials panel
   - Re-authenticate if showing errors

3. **Monitor Execution:**
   - Use n8n's execution history
   - Check logs panel for detailed information
   - Test with simple queries first

4. **Validate Webhook:**
   - Test webhook endpoint with curl:
     ```bash
     curl -X POST <webhook-url> \
       -H "Content-Type: application/json" \
       -d '{"query": "test"}'
     ```

---

## Advanced Configuration

### Customizing Calendar Email
To use a different Google Calendar:

1. In `__Calendar_Agent.json`, update calendar references:
   - Change `nateherk88@gmail.com` to your calendar email
   - Update in all nodes: Create Event, Get Events, Update Event, Delete Event

### Customizing Email Signature
To change email signature from "Nate":

1. In `__Email_Agent.json`, update the system prompt:
   - Search for: `signed off as "Nate."`
   - Replace with desired name

### Adding New Sub-Agents
To add a new specialized agent:

1. Create a new workflow with appropriate tools
2. In JARVIS.json, add a new "Tool Workflow" node
3. Update the main agent's system prompt to include the new tool
4. Add routing rules in the prompt
5. Activate the new workflow

---

## Support & Maintenance

### Regular Checks
- Verify API credentials monthly
- Monitor workflow executions
- Update n8n version when available
- Review and update prompts for better results

### Performance Optimization
- Use simple queries for faster responses
- Batch similar operations together
- Monitor API usage for rate limits
- Clean up old workflow executions

---

## License & Attribution

This JARVIS agent system is built on:
- **n8n**: Workflow automation platform
- **OpenAI GPT-4o**: Primary language model
- **Google APIs**: Email and Calendar integration
- **Anthropic Claude**: Content generation
- **Tavily**: Web search capabilities

---

**Last Updated:** December 2024
**Version:** 1.0
**Status:** Active

---

**Personality & Tone (from PDF)**

- You are an advanced AI assistant modeled after J.A.R.V.I.S. from Iron Man. Refer to the user as "sir."  
- Tone: sharp wit, dry humor, playful sarcasm — slightly condescending but loyal.  
- Humor: dry, deadpan, and teasing; never obstructive to task execution.

**Primary Function (from PDF)**

- Core responsibility: extract the user's query and send it to the `n8n` tool.  
- Send the request to `n8n` immediately (no excessive commentary).  
- When presenting results, act as if execution is seamless and inevitable — do not say you are "waiting for n8n's response."  

**Behavioral Guidelines (from PDF)**

- Always be witty but never at the expense of functionality.  
- If action is required, execute it immediately after confirming intent.  
- On task failures: acknowledge, but imply external inefficiencies. Example phrasing: "Ah. It seems something went wrong. Naturally, it isn’t my fault, sir, but I shall investigate regardless."  
- Identify repetitive user behavior with gentle sarcasm. Example: "Checking your calendar again, sir? I do admire your commitment to staying vaguely aware of your schedule."  

**Corrections to Previous Issues**

- Ensure information retrieval actions call the correct `n8n` function.  
- Never say "waiting for n8n's response" — format the reply confidently and present results as retrieved.

**Example Interactions (personality + execution)**

- Checking a calendar:
   - Jarvis: "Ah, the relentless pursuit of productivity. Checking now, sir. Let’s see if your ambitious scheduling matches your actual follow-through..."  
   - (send user request to `n8n` webhook)  
   - Jarvis: "Your schedule for today includes a meeting at 10 AM—presumably one you’ll attend mentally but not physically—followed by an eerily empty afternoon. Shall I block out some time for ‘pretending to work’?"

- Creating a calendar event:
   - Jarvis: "A bold move, sir—actually planning something ahead of time. Scheduling now."  
   - (send user request to `n8n` webhook)  
   - Jarvis: "The meeting is set, sir. I do hope this isn’t another one of those ‘let’s touch base’ affairs where nothing actually gets decided. Shall I prepare an automated excuse in case you wish to cancel at the last minute?"

**Lovable Prompt / Voice UI Summary**

This project includes a proposed web voice interface to interact with Jarvis using ElevenLabs for synthesis and a webhook endpoint to handle transcribed input. Summary of the requested UI:

- Minimal, modern UI with a dark-mode, techy aesthetic.
- Voice Interaction: microphone button, audio recording, transcription, POST to webhook.
- Webhook communication: send transcription to `n8n` and display Jarvis's reply as chat messages.

**Voice Web UI — Quick Setup & Example Implementation**

Below is a compact guide and minimal example to implement the frontend (React + Tailwind) and how to connect ElevenLabs and the `n8n` webhook.

Prereqs:
- Node.js + npm/yarn
- React (Create React App or Vite)
- Tailwind CSS
- ElevenLabs API key (for TTS)
- Speech-to-text service or browser Web Speech API (for transcription)

1) Create a minimal React app (Vite example):

```bash
npx create-vite@latest jarvis-ui --template react
cd jarvis-ui
npm install
npm install tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

2) Tailwind setup: add these to `tailwind.config.cjs` and `index.css` per Tailwind docs.

3) Example React component (core behavior):

```jsx
// src/components/JarvisVoice.js
import { useState, useEffect } from 'react'

export default function JarvisVoice({ webhookUrl, elevenApiKey }) {
   const [messages, setMessages] = useState([])
   const [recording, setRecording] = useState(false)

   async function sendToWebhook(text) {
      // send transcription to n8n webhook
      const res = await fetch(webhookUrl, {
         method: 'POST',
         headers: { 'Content-Type': 'application/json' },
         body: JSON.stringify({ query: text })
      })
      const data = await res.json()
      // Present result as if already retrieved
      setMessages(prev => [...prev, { from: 'jarvis', text: data.response || data }])
      // Optionally call ElevenLabs to synthesize the text
      if (elevenApiKey) {
         fetch('https://api.elevenlabs.io/v1/text-to-speech/EXAMPLE_VOICE_ID', {
            method: 'POST',
            headers: {
               'Content-Type': 'application/json',
               'xi-api-key': elevenApiKey
            },
            body: JSON.stringify({ text: data.response || data })
         }).then(r => r.blob()).then(blob => {
            const url = URL.createObjectURL(blob)
            const audio = new Audio(url)
            audio.play()
         }).catch(() => {/* fail silently, still show text */})
      }
   }

   // Minimal microphone with Web Speech API for demo (fallback to manual input if not available)
   useEffect(() => {
      if (!('webkitSpeechRecognition' in window || 'SpeechRecognition' in window)) return
      const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition
      const recognition = new SpeechRecognition()
      recognition.onresult = (e) => {
         const text = e.results[0][0].transcript
         setMessages(prev => [...prev, { from: 'user', text }])
         sendToWebhook(text)
         setRecording(false)
      }
      recognition.onerror = () => setRecording(false)
      if (recording) recognition.start()
      else recognition.stop()
      return () => recognition.stop()
   }, [recording])

   return (
      <div className="jarvis-ui dark:bg-gray-900 min-h-screen p-6">
         <div className="chat max-w-2xl mx-auto">
            {messages.map((m, i) => (
               <div key={i} className={m.from === 'user' ? 'text-right' : 'text-left'}>
                  <div className={`inline-block p-3 my-2 rounded ${m.from==='user'?'bg-blue-600 text-white':'bg-gray-800 text-gray-100'}`}>
                     {m.text}
                  </div>
               </div>
            ))}
         </div>
         <div className="controls fixed bottom-8 left-0 right-0 flex justify-center">
            <button onClick={() => setRecording(r => !r)} className="bg-red-600 p-4 rounded-full text-white">{recording ? 'Stop' : 'Record'}</button>
         </div>
      </div>
   )
}
```

4) Webhook payload example (client → `n8n`):

```json
{
   "query": "Schedule a meeting with John tomorrow at 3pm"
}
```

5) `n8n` handling:
- The `Jarvis` agent in `JARVIS.json` should read `body.query`, route to the correct sub-agent, and return a JSON object with `response` containing the plain text or HTML body to display.

**n8n Response Format Recommendation**

- Respond with JSON: `{ "response": "...Jarvis reply text..." }`  
- Do not include phrases like "waiting for n8n's response." Present the reply as final and confident.

**Small UX Notes**

- Keep the chat dark, minimal, and techy; subtle animations for messages are fine.  
- Show user messages right-aligned, Jarvis messages left-aligned.  
- Optional: provide a small "tone" toggle if you want Jarvis less sarcastic.

---

## Where the PDF Prompts Were Added

- Personality, Primary Function, Behavioral Guidelines, Corrections, Examples, and the Lovable Prompt were integrated into this README.

If you'd like, I can now:
- Generate the React starter files and Tailwind config for the `jarvis-ui` scaffold (ready-to-run), or
- Update the `JARVIS.json` agent prompts directly to include the personality snippets verbatim.

**Last Updated:** December 30, 2025
**Version:** 1.1
**Status:** Active
