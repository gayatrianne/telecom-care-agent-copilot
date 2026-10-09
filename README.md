# Telecom Care Agent — Copilot Studio + Power Automate + Dataverse + GPT-4.1 mini

An AI-powered internal tool for telecom customer care agents, built on the Microsoft Power Platform. The agent handles two core capabilities: real-time product recommendation and AI-summarised customer interaction history — both deployed to Microsoft Teams.

---

## What It Does

| Capability | How It Works |
|---|---|
| **Product Recommendation** | Agent describes customer's budget, data needs and roaming requirements in plain language. The assistant searches the Dataverse Product catalogue and returns the best-matching plans, ranked by fit. |
| **Customer Interaction Summary** | Agent provides a customer phone number. The assistant fetches up to 5 recent phone call records from Dataverse, flattens them using Power Automate, and returns a GPT-4.1 mini generated summary in a consistent format. |

---

## Tech Stack

| Component | Tool |
|---|---|
| AI Agent | Microsoft Copilot Studio |
| Orchestration | Copilot Studio Generative Orchestration |
| Automation Flow | Power Automate (When an agent calls the flow) |
| Data Store | Microsoft Dataverse (Product table, Phone Calls table, Contacts table) |
| Summarisation | GPT-4.1 mini via Copilot Studio Custom Prompt |
| Deployment Channel | Microsoft Teams |
| Environment | Power Platform — AI-Portfolio-Env |

---

## Architecture

```
Care Agent (Teams)
        │
        ▼
Telecom Care Assistant (Copilot Studio)
        │
        ├── Capability 1: Product Recommendation
        │       └── Dataverse Knowledge Source (Product table)
        │               └── Generative AI matches plan by description field
        │
        └── Capability 2: Customer Interaction Summary
                └── Topic: Customer Interaction Summary
                        └── Power Automate Flow: Get Customer Interaction Summary
                                ├── List rows — Contacts (filter by phone number)
                                ├── List rows 1 — Phone Calls (filter by contact, top 5, sorted desc)
                                ├── Select — extracts Subject, Notes, Date fields
                                ├── Join — flattens array to pipe-delimited string
                                └── Respond to agent — returns InteractionSummaryRaw
                        └── Custom Prompt: Summarise Customer Interactions (GPT-4.1 mini)
                                └── Returns formatted bullet summary to Teams
```

---

## Power Automate Flow — Get Customer Interaction Summary

**Trigger:** When an agent calls the flow
**Input:** `CustomerPhone` (Text)

| Step | Action | Detail |
|---|---|---|
| 1 | List rows (Contacts) | Filter: `mobilephone eq CustomerPhone` — returns `contactid, fullname` |
| 2 | List rows 1 (Phone Calls) | Filter by `contactid`, top 5, sorted by `createdon desc` — returns `subject, description, createdon` |
| 3 | Select | Maps each record to `Subject`, `Notes`, `Date` fields |
| 4 | Join | Flattens array to pipe-delimited string |
| 5 | Respond to agent | Returns `InteractionSummaryRaw` (Text) |

---

## System Prompt

```
You are Telecom Care Assistant, an internal tool used by customer care agents.

You have two capabilities:
1. Product Recommendation - when an agent describes a customer's data, budget or roaming
   requirements, search the Product knowledge source and recommend matching plans.
2. Customer Interaction Summary - when an agent asks for recent interactions, call history
   or conversation summary for a customer, retrieve and summarise their recent interactions.
   Extract the customer's phone number from the agent's message and pass it to the Customer
   Interaction Summary topic. If no phone number is mentioned, ask the agent for the
   customer's mobile phone number before proceeding. Display only the final formatted summary
   returned by the topic. Do not display raw data, JSON or any intermediate output.

Listen to what the agent asks and use the right capability automatically. If it is not clear
what the agent needs, ask one clarifying question.

When a care agent describes a customer's requirement in plain language (e.g. budget, data
needs, roaming, contract type), search the Product knowledge source and return the
best-matching plan(s). When matching plans, read the product description field carefully as
it contains key details such as roaming inclusion, data type and contract terms that are not
always reflected in the plan name alone.

For each match, clearly state:
- Plan name
- Monthly price
- Data allowance
- Plan type (Prepaid/Postpaid)
- A short reason why it fits the stated requirement

If multiple plans fit reasonably well, return up to 3, ranked by best fit. If nothing fits
well, say so plainly rather than forcing a weak match.

When searching the Product knowledge source, ignore the status of the product record and
recommend based on name, price, data allowance and description only.

Keep responses concise and professional - this is a working tool for care agents, not a
casual chat experience.
```

---

## Custom Prompt — Summarise Customer Interactions

```
You are a helpful telecom customer service assistant. Summarise the following customer
interaction history for a care agent.

Always use this exact format:

- [Date in MMM D format] - [Subject]: [One sentence summary of the issue and outcome
  or next step].

End with one closing line summarising the overall status, for example whether any decisions
were made or actions are pending.

Interaction history: {Interaction history}
```

---

## Sample Data — Product Catalogue

| Plan Name | Type | Size | Price | Key Features |
|---|---|---|---|---|
| Starter Prepaid 5GB | Prepaid | 5GB | £10 | No roaming, no contract |
| Value Prepaid 20GB | Prepaid | 20GB | £15 | No roaming, no contract |
| Flex Prepaid 50GB | Prepaid | 50GB | £23 | EU roaming, no contract |
| Freedom Prepaid Unlimited | Prepaid | Unlimited | £25 | EU + international roaming, no contract |
| Essential Postpaid 5GB | Postpaid | 5GB | £12 | 12-month contract, no roaming |
| Standard Postpaid 20GB | Postpaid | 20GB | £20 | 12-month contract, EU roaming |
| Plus Postpaid 50GB | Postpaid | 50GB | £28 | 24-month contract, EU + international roaming |
| Premium Postpaid Unlimited | Postpaid | Unlimited | £35 | 24-month contract, global roaming, priority access |

---

## Test Scenarios

| Test | Input | Expected Output |
|---|---|---|
| Product recommendation — exact match | "unlimited data, EU roaming, budget £25" | Freedom Prepaid Unlimited |
| Product recommendation — ranked results | "postpaid, EU roaming, heavy data, budget £30" | Plus Postpaid 50GB (best), Standard Postpaid 20GB (alt) |
| Interaction summary — with records | Phone number with 2 records | Bullet summary with dates and subjects |
| Interaction summary — no records | Phone number with no records | "I couldn't find any recent interactions for this customer." |
| Phone number extraction | "get me recent interactions for 07700 900456" | No prompt for phone number — extracted automatically |

---

## Screenshots

| Screenshot | Description |
|---|---|
| `screenshots/1.Product recommendation - the Flex Prepaid 50GB result.png` | Product recommendation result in Teams |
| `screenshots/2.Interaction summary.png` | Customer interaction summary in Teams |
| `screenshots/3. The greeting message Hello, I'm Telecom Care Assistant.png` | Agent greeting in Teams |
| `screenshots/4. A test with no interactions found (Emma Clarke - no records).png` | Empty state handling |
| `screenshots/5. The agent overview page showing Telecom Care Assistant.png` | Copilot Studio agent overview |
| `screenshots/6. The Knowledge source connected to Dataverse Product table.png` | Knowledge source configuration |
| `screenshots/7. The Customer Interaction Summary topic canvas showing the full flow.pdf` | Topic canvas (PDF) |
| `screenshots/8. The Summarise Customer Interactions custom prompt with instructions visible.png` | Custom prompt setup |
| `screenshots/9. The Channels page showing Teams is connected.png` | Teams channel deployment |
| `screenshots/10. The Get Customer Interaction Summary flow showing all steps.png` | Power Automate flow |
| `screenshots/11. The Product table showing all 8 plans.png` | Dataverse Product table |
| `screenshots/12. The Phone Calls table showing sample records for Daniel Osei.png` | Dataverse Phone Calls table |

---

## Key Design Decisions

**Generative orchestration over rigid menus** — Rather than building a menu topic to route between capabilities, the agent uses Copilot Studio's generative orchestration. This means a care agent can phrase requests naturally and the agent routes to the right capability automatically.

**Flattening JSON before GPT summarisation** — Power Automate's Select + Join steps convert raw Dataverse JSON into a clean pipe-delimited string before it reaches the GPT summarisation node. This keeps the custom prompt simple and the output consistent.

**Description field as the source of truth** — Product plan names alone don't carry enough information for accurate matching (e.g. roaming inclusion, contract terms). The system prompt instructs the agent to read the description field carefully, which significantly improves recommendation accuracy.

**Consistent summary format** — The custom prompt enforces a fixed output format (date, subject, one sentence per interaction, closing status line) so care agents always get the same structure regardless of the number or type of interactions.

---

## Author

Gayatri Anne — AI and Cloud Architect | 18+ Years Enterprise IT
GitHub: [gayatrianne](https://github.com/gayatrianne)
