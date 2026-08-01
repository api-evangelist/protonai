---
name: Capture a lead and open a CRM opportunity
description: Use the Proton CRM endpoints to record a new lead, promote it to a customer and contact, open a pipeline opportunity, and log the first call note.
api: openapi/protonai-openapi.yml
operations: [createLeadV1, createCustomerV1, createContactV1, createOpportunity, createCallNoteV1]
---

# Capture a lead and open an opportunity

This flow turns an inbound lead into tracked CRM activity for a distributor tenant. All requests go to `https://api.proton.ai/{company}/...` with `X-Api-Key`, `X-Company` (required) and optional `X-User-Id` headers. Writes are not idempotent — do not blind-retry a `createOpportunity` on a timeout without first searching for the created record.

## Steps

1. **Record the lead** — `createLeadV1` (`POST /{company}/v1/leads`) with the company name, contact name, and any email/phone. Returns a lead with `id` and `customer_id`.
2. **Promote to customer** — `createCustomerV1` (`POST /{company}/v1/customers`) once the lead is qualified, to create the billing/customer record.
3. **Add the contact** — `createContactV1` (`POST /{company}/v1/contacts`) attaching the person to the `customer_id`.
4. **Open the opportunity** — `createOpportunity` (`POST /{company}/opportunities`) with `pipeline_id`, `stage_id`, `title`, `value`, `customer_id`, and `contact_id`.
5. **Log the first touch** — `createCallNoteV1` (`POST /{company}/v1/call_notes`) to record the initial conversation against the customer/contact.

## Notes

- Responses use the `{ "status_message": "...", "data": { ... } }` envelope; `id` (UUID) and the human `customer_id`/`contact_id` are distinct — reference the domain ids when linking records (see `data-model/protonai-data-model.yml`).
- Retrieve pipeline/stage ids from `getLeadStagesV1` and pipeline endpoints before creating an opportunity.
