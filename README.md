# 🌍 JenTravel — Agent-Powered Travel Assistant

**JenTravel** is a fictional travel agency based in **Lebanon**.  
It uses **OpenAI’s GPT model with function calling** and a **Gradio front-end** to simulate a professional digital travel concierge capable of answering customer inquiries, collecting leads, and recording unanswered questions for follow-up.

---

## Overview

JenTravel acts as a smart representative of a travel company, using real company data from:
- `about_business.pdf` — detailed brochure with destinations, services, and partnerships  
- `summary.txt` — detailed available packages and visa information  

The chatbot reads this data, stays strictly within it, and engages customers naturally.  
If it doesn’t know something, it logs the question for future review instead of guessing.

---

## Features

- **Context-aware chat** — remembers previous turns and responds consistently  
- **Two functional tools:**  
| Tool | Description | Output |
|------|--------------|---------|
| `record_customer_interest(email, name, message)` | Saves leads (name, email, message) to a JSONL log | `data/leads.jsonl` |
| `record_feedback(question)` | Records questions that the bot cannot answer | `data/feedback.jsonl` |

- **Business knowledge grounding** — reads from real PDF and TXT files  
- **Beautiful Gradio UI** — custom teal-cyan theme with gradient header and chat avatars  
- **Runs seamlessly in Google Colab** — creates a public Gradio share link

---

## Tools

The application has 2 working tools:

###  1. `record_customer_interest(email, name, message)`

**Purpose:**  
Captures and logs customer leads who express interest in booking or learning more about a package.

**How it works:**  
Whenever a user shares contact information (e.g., name and email) or expresses booking interest (“I’d like to book a trip”, “Can you contact me?”), the assistant automatically triggers this tool.

**What it does:**  
- Saves the customer’s name, email, and inquiry message into a file named: data/leads.jsonl
- Each entry is stored as a structured JSON line for easy retrieval.
- This allows human agents or marketers to follow up with real customers later.

### 2. `record_feedback(question)` 

**Purpose:**  
Logs user questions or requests that fall outside the assistant’s available knowledge.

**How it works:** 
When a user asks about topics not covered in summary.txt or about_business.pdf
(for example, “Do you offer travel insurance?” or “What are your refund policies?”),
the assistant does not guess — it calls this tool instead.

**What it does:** 
- Saves the exact question text in a JSONL file: data/feedback.jsonl
- This creates a structured record of gaps in customer inquiries — allowing the business to expand or clarify its offerings in the future.

## Architecture

```
/content/
│
├── business_agent.ipynb         # Main Colab notebook
├── about_business.pdf           # Full business profile
├── summary.txt                  # Business summary and packages
├── requirements.txt             # Project dependencies
│
└── data/
    ├── leads.jsonl              # Saved customer leads
    └── feedback.jsonl           # Logged unanswered questions
```

---

## Logic Flow

1. When the user asks about **known destinations or services**, JenTravel answers using the PDF + TXT content.  
2. If the user expresses **interest or provides contact info**, the assistant automatically calls `record_customer_interest` and it saves the email to a file so that assistants can contact the customer.  
3. If the user asks about **unsupported topics** (e.g., insurance, policies), the assistant calls `record_feedback` to save the answes so that the agency adds it to its Info.  
4. All responses are polite, concise, and tailored for Lebanese travelers.

---


## How to Run in Colab


1. **Add your OpenAI API key**
   You can add your key in one of two ways:
   - **Option 1:** Create a `.env` file in Colab and include:  
     ```env
     OPENAI_API_KEY=your_api_key_here
     ```

     Then load it:
     ```from dotenv import load_dotenv
      load_dotenv()
      ```

   - **Option 2:** Set it directly in your Colab environment:  
     ```python
     import os
     os.environ["OPENAI_API_KEY"] = "your_api_key_here"
     ```
   This is needed because the code takes this from environment
   client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

2. Insatll requirements
   ```
   !pip install -r requirements.txt
   ```

3. **Run all cells**  
   Once you execute all cells in the notebook, a **public Gradio URL** will be automatically generated.  
   You can open this link in your browser to **interact live with the JenTravel assistant** 


---


## Example Conversation (Demo)

Below is a real usage flow demonstrating how **JenTravel** interacts with customers — using its internal knowledge, tools, and function-calling logic.

---

**User:**  
What packages do you have for Turkey?

**Bot:**  
We currently have the following package for Turkey:

**Cappadocia Adventure — Turkey (4 Days)**  
- Hot-air balloon experience and cave hotel stay  
- Guided cultural tours in Göreme and Urgup  
- Direct flights via Istanbul to Kayseri  

If you're interested in this package or need more information, please share your name and email!

---

**User:**  
I am interested in this package.

**Bot (calls → `record_customer_interest`)**  
Could you please provide me with your name and email so I can note your interest in the package? Thank you!

> 🧩 **Behind the scenes:**  
> When the assistant detects interest or booking intent, it automatically **calls the function** `record_customer_interest(email, name, message)`.  
> This creates a structured log entry inside:
> ```
> data/leads.jsonl
> ```
> The file acts as a digital “lead book,” where each line is a JSON record containing the customer’s contact info and inquiry message.

---

**User:**  
Jenny Haddad, jenny@gmail.com

**Bot:**  
Thank you, Jenny! I've noted your request regarding the prices for our travel packages.  
A travel advisor will reach out to you shortly with more information.  
If you have any other questions or need assistance, feel free to ask!

> 🧩 **Behind the scenes:**  
> The assistant saves this new entry:
> ```json
> {"email": "jenny@gmail.com", "name": "Jenny Haddad", "message": "Interested in Cappadocia Adventure package"}
> ```

---

**User:**  
Do you offer help with visa?

**Bot:**  
Yes, we offer visa assistance for various destinations, including **Schengen, UK, GCC, and Asia**.  
Our partnership with the **TLS Visa Application Center** ensures a smooth visa processing experience.  
If you're interested in more details, please share your name and email!

---

**User:**  
Do you offer travel insurance?

**Bot (calls → `record_feedback`)**  
I've noted your question about travel insurance for follow-up.  
If you're interested, please share your name and email, and I'd be happy to record your interest!

> 🧩 **Behind the scenes:**  
> Since “travel insurance” isn’t found in the PDF or TXT knowledge base,  
> the model calls the `record_feedback(question)` tool, which saves the user’s question to:
> ```
> data/feedback.jsonl
> ```
> ```json
> {"question": "Do you offer travel insurance?"}
> ```
> This ensures unanswered questions are captured and can later be addressed by the travel agency.

---





---

