![AI-ERP — Automation, AI Agents and RAG with n8n](docs/assets/ai-erp-hero.svg)

[![n8n](https://img.shields.io/badge/Automation-n8n_Cloud-EA4B71?style=flat-square&logo=n8n&logoColor=white)](https://n8n.io)
[![AI Agent](https://img.shields.io/badge/Pattern-AI_Agent-6657E8?style=flat-square)](#the-ai-side)
[![RAG](https://img.shields.io/badge/Knowledge-RAG-16A394?style=flat-square)](#the-ai-side)
[![Airtable](https://img.shields.io/badge/Data-Airtable-18BFFF?style=flat-square&logo=airtable&logoColor=white)](https://airtable.com)
[![Telegram](https://img.shields.io/badge/Interface-Telegram-26A5E4?style=flat-square&logo=telegram&logoColor=white)](https://core.telegram.org/bots)
[![Gmail](https://img.shields.io/badge/Sales-Gmail_API-EA4335?style=flat-square&logo=gmail&logoColor=white)](#the-automations)
[![Google Drive](https://img.shields.io/badge/Documents-Google_Drive-34A853?style=flat-square&logo=googledrive&logoColor=white)](#the-automations)

# AI-ERP — Automation, AI Agents & RAG with n8n

A working mini-ERP for a fictional Israeli cosmetics & skincare business, automated end to end with **n8n Cloud**, **AI agents** and **RAG**. No servers, no Docker, nothing installed — the whole system is n8n Cloud plus Airtable. Built as a training project; every workflow is runnable and demoable.

**[עברית ↓](#ai-erp--אוטומציה-סוכני-ai-ו-rag-עם-n8n)**

---

### How it works

Airtable is the database (8 tables) and the UI. Documents are rendered to PDF and **stored in Google Drive** — never in the database. Chat and embeddings run on separate OpenAI-compatible endpoints set in n8n credentials, so both are swappable; the embeddings model is multilingual so Hebrew works. n8n Cloud supplies the public HTTPS URL, so the Telegram bots need no tunnel.

![AI-ERP workflow architecture: Telegram, Gmail and schedule triggers feed n8n Cloud agents and RAG, which write out to Airtable, Gmail, Google Drive and Telegram](docs/assets/architecture-flow.svg)

<details>
<summary>Plain-text version of the diagram</summary>

```
Telegram (manager)  ──┐                    ┌──▶ Airtable       (data)
Telegram (support)  ──┤                    ├──▶ Gmail          (sales email)
Gmail inbox         ──┼──▶  n8n Cloud  ───┼──▶ Google Drive   (documents + PDFs)
Schedules           ──┘  agents + RAG     └──▶ Telegram       (replies)
```

</details>

### The automations

| #  | Workflow               | Trigger                            | Demonstrates                                                                                            |
| --- | ---------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------- |
| 1  | Tax-Doc Validation     | new Invoice / TaxInvoice / Receipt | Israeli VAT rules (18%, 17% before 2025), sequential doc numbers; valid → file queue, invalid → flagged |
| 3  | Contact Intake         | new Lead                           | Normalisation + de-duplication by email                                                                 |
| 4a | Sales Cold Emails      | every 3h                           | LLM writes a personalised email, Gmail sends it, lead marked contacted                                  |
| 4b | Sales Reply Check      | Gmail                              | Parse the reply, match the lead, agent drafts and sends an answer                                       |
| 5  | Customer Service Agent | Telegram                           | RAG agent — policies + products as retrieval tools, Hebrew and English                                  |
| 6  | Policies Embedding     | manual                             | Documents → chunks → embeddings → vector store                                                          |
| 7  | Products Embedding     | manual                             | Same pipeline over the live catalogue                                                                   |
| 8  | Document Pipeline      | every minute                       | Queue → RTL HTML → PDF → **Google Drive**                                                               |
| 9  | Manager Agent          | Telegram                           | Tool calling (`search_invoices`, `search_tasks`, `create_task`), owner-only                             |
| 13 | UI Chat Webhook        | webhook                            | Agent answers over an HTTP tool, synchronous response                                                   |

### The AI side

- **Manager agent** — calls tools, can create records. Sender is authorised before the agent runs.
- **Support agent** — pure RAG, so it can't invent a return policy.
- **Sales agent** — one workflow writes and sends, another watches the inbox and replies.

RAG: policy docs and the product catalogue are chunked, embedded and queried by the agents as a retrieval tool at answer time.

### Stack

`n8n Cloud` · `AI agents & tool calling` · `RAG / vector store` · `Airtable` · `Telegram Bot API` · `Gmail + Drive APIs` · `OAuth 2.0`

### Known limits

Vector store and chat memory are in-memory (wiped on restart). No filesystem on Cloud, so policy documents and templates come from Drive or live inside the workflow. Airtable polls at ≥1 minute off a `Created` field. Sequential numbering can race inside one poll window. Relationships are string ids (`CUST-0001`), not links.

---

---

# AI-ERP — אוטומציה, סוכני AI ו-RAG עם n8n

מערכת ERP קטנה ועובדת לעסק קוסמטיקה וטיפוח ישראלי (פיקטיבי), מאוטמת מקצה לקצה עם **n8n Cloud**, **סוכני AI** ו-**RAG**. בלי שרתים, בלי Docker, בלי שום התקנה — הכול n8n Cloud ו-Airtable. פרויקט לימודי; כל workflow רץ וניתן להדגמה.

**[English ↑](#ai-erp--automation-ai-agents--rag-with-n8n)**

---

### איך זה עובד

Airtable הוא בסיס הנתונים (8 טבלאות) וגם הממשק. מסמכים מרונדרים ל-PDF ו**נשמרים ב-Google Drive** — לא בבסיס הנתונים. השיחה והאמבדינגס רצים על נקודות קצה נפרדות תואמות OpenAI שמוגדרות בקרדנצ׳יאלס של n8n, ולכן ניתנות להחלפה; מודל האמבדינגס רב-לשוני כדי שעברית תעבוד. n8n Cloud מספק כתובת HTTPS ציבורית, ולכן הבוטים בטלגרם לא צריכים מנהרה.

![תרשים ארכיטקטורת AI-ERP](docs/assets/architecture-flow.svg)

<details>
<summary>גרסת טקסט פשוט של התרשים</summary>

```
טלגרם (מנהל)   ──┐                     ┌──▶ Airtable       (נתונים)
טלגרם (שירות)  ──┤                     ├──▶ Gmail          (מיילי מכירות)
תיבת Gmail      ──┼──▶  n8n Cloud  ────┼──▶ Google Drive   (PDF של מסמכים)
תזמונים         ──┘  סוכנים + RAG      └──▶ Telegram       (תשובות)
```

</details>

### האוטומציות

| #  | Workflow          | טריגר                            | מה זה מדגים                                                                     |
| --- | ----------------- | -------------------------------- | ------------------------------------------------------------------------------- |
| 1  | בדיקת מסמכי מס    | חשבונית / חשבונית מס / קבלה חדשה | חוקי מע״מ ישראלי (18%, 17% לפני 2025), מספור רץ; תקין ← תור קבצים, פסול ← מסומן |
| 3  | קליטת אנשי קשר    | ליד חדש                          | נרמול וזיהוי כפילויות לפי אימייל                                                |
| 4a | מיילי מכירות קרים | כל 3 שעות                        | LLM מנסח מייל מותאם, Gmail שולח, הליד מסומן                                     |
| 4b | בדיקת תשובות      | Gmail                            | פירוק התשובה, התאמה לליד, סוכן מנסח ושולח מענה                                  |
| 5  | סוכן שירות לקוחות | טלגרם                            | סוכן RAG — מדיניות ומוצרים ככלי שליפה, עברית ואנגלית                            |
| 6  | אמבדינג מדיניות   | ידני                             | מסמכים ← צ׳אנקים ← אמבדינג ← vector store                                       |
| 7  | אמבדינג מוצרים    | ידני                             | אותו צינור מעל הקטלוג החי                                                       |
| 8  | צינור מסמכים      | כל דקה                           | תור ← HTML בעברית RTL ← PDF ← **Google Drive**                                  |
| 9  | סוכן מנהל         | טלגרם                            | קריאה לכלים (`search_invoices`, `search_tasks`, `create_task`), לבעלים בלבד     |
| 13 | Webhook צ׳אט      | webhook                          | סוכן עונה דרך כלי HTTP, תשובה סינכרונית                                         |

### הצד של ה-AI

- **סוכן מנהל** — קורא לכלים, יכול ליצור רשומות. השולח מאומת לפני שהסוכן רץ.
- **סוכן שירות** — RAG טהור, ולכן לא ממציא מדיניות החזרות.
- **סוכן מכירות** — workflow אחד מנסח ושולח, שני עוקב אחרי התיבה ומשיב.

RAG: מסמכי המדיניות והקטלוג נחתכים לצ׳אנקים, עוברים אמבדינג, והסוכנים מתשאלים אותם ככלי שליפה בזמן המענה.

### סטאק

`n8n Cloud` · `סוכני AI וקריאה לכלים` · `RAG / vector store` · `Airtable` · `Telegram Bot API` · `Gmail + Drive APIs` · `OAuth 2.0`

### מגבלות ידועות

ה-vector store וזיכרון השיחה בזיכרון בלבד (נמחקים באתחול). ב-Cloud אין מערכת קבצים, ולכן מסמכי המדיניות והתבניות מגיעים מ-Drive או יושבים בתוך ה-workflow. Airtable מתשאל כל דקה לפחות לפי שדה `Created`. מספור רץ עלול להתנגש באותו חלון polling. הקשרים הם מזהי מחרוזת (`CUST-0001`), לא קישורים.
