# AI-Supported Ticket Management

English portfolio for a university team project at the University of Duisburg-Essen. The team built a support assistant for the ZIM IT service desk. A person describes a request in a chat. A language model proposes one ITSM ticket category. [Zammad](https://zammad.com/) stores that category on the ticket. A support agent then reviews the ticket and can keep, change, or clear the category.

This repository is a reviewed summary. It does not republish the application source, deployment configuration, credentials, or records from real support tickets. Examples in the [case study](docs/case-study.md) are fictional.

## What the team built

The assistant collects the request, classifies it, and can open a ticket in Zammad. The category step is one part of that flow:

1. The model reads the whole conversation, the issue description, and any extra details already collected.
2. It must choose exactly one of **Incident**, **Service Request**, **Change**, **Problem**, or **Complaint**.
3. If the returned label is not one of those five, the system stores **Service Request** instead.
4. The chosen value is written to the Zammad ticket field `kategorie` (shown to agents as **Kategorie**) when the ticket is created or when its metadata is finalized. The same value is also copied into the ticket article.
5. On an open support ticket, the support agent sees that field and decides whether it stays. The model does not send the support reply and does not decide that the request is resolved.

A separate path can close a ticket as solved by the chatbot, without a support agent. Human review of the suggested category happens on tickets that remain open for the service desk. The article text keeps the category from the moment it was written. A later change to the select field does not rewrite that article.

The diagram and the hand-off are in [Ticket triage](docs/architecture.md). A fictional walkthrough is in the [case study](docs/case-study.md).

## Team

This was joint work. The implementation history records contributions from Michail Cheklin, Faruk-Emre Ertas, Ikram Hulich, Firas Khamis, Peter Krauza, Yvan Steve Tague Sokoudjou, and Max Unterbusch. Names are taken from that history, not from a separate roster.

## Personal contribution

Only work that appears under the git authors `taguesokoudjou` and `Tague Sokoudjou, Yvan Steve` is listed here:

- Added ITSM ticket-category classification, including its documentation and the split between unit tests and live tests.
- Moved the category configuration out of the graph node module.
- Showed the classified category on the Zammad ticket as a customer-visible field that an agent can edit.
- Added the one-time email transmission that runs when a public support-staff article is added, together with the related tests and CI wiring.
- Also committed supporting work on incident escalation, ticket creation, the local container setup, and code cleanup. Those commits do not make the rest of the chatbot, retrieval, or interface a solo effort.

## What this portfolio leaves out

Production tickets, student identifiers, mailboxes, API tokens, and internal host names are not included. The category rules on the triage page are a short English summary of the classifier, not a copy of the operational prompt.
