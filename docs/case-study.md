# Case study: a suggested category and the agent's decision

Every name, address, and ticket below is fictional. Nothing in this page comes from a real Zammad ticket.

## Request

Alex Example writes in the support chat:

> I cannot connect to the campus VPN from home. The client stops at the login step. I need it for class today.

The assistant has already stored these fictional details:

| Item | Value |
| --- | --- |
| Name | Alex Example |
| Email | alex.example@example.edu |
| Extra detail | The failure started today and affects this one login |
| Main intent | Restore a service that currently does not work |

There is no complaint about waiting time, no request to change a permission, and no ask to investigate a recurring root cause.

## What the model suggests

The classifier reads the whole conversation, not only the last sentence. VPN access is failing now, so the matching label is **Incident**. The overlap order would have preferred Complaint, Problem, Change, or Service Request if one of those had been the main intent. None of them is.

The system writes `Incident` to the Zammad field `kategorie` and copies `Kategorie: Incident` into the new ticket article. The ticket stays open for the service desk.

## Where the agent reviews it

Support agent Jordan Example opens the ticket in Zammad.

- The **Kategorie** field shows Incident. Jordan can keep it, choose another of the five categories, or clear it.
- The article still shows the original line even if Jordan changes the field later.
- Jordan keeps Incident, because this is a current loss of service for one person, not a known problem record under root-cause analysis.
- Jordan writes the support reply. The model does not send that reply and does not mark the request resolved.

That review is the support team's decision. The stored category is the suggestion that reached the agent.

## A label the system would not store

If the model returned `VPN` instead of one of the five categories, the ticket would not keep that word. The writer accepts only Incident, Service Request, Change, Problem, and Complaint. An unknown label is replaced with **Service Request** before Zammad is updated. Jordan would still be able to change that fallback after opening the ticket.

## Related pages

- [Project overview](../README.md)
- [Ticket triage](architecture.md)
