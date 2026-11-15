Retell AI Voice Agent: Appointment Workflows (n8n + Cal.com)

Two n8n workflows that let a Retell AI voice agent cancel or reschedule Cal.com bookings by calling n8n webhooks as custom functions.

File	Webhook path	What it does
appointment_cancellation_sanitized.json	POST /webhook/cancel	Cancels the caller's upcoming booking
appointment_rescheduling_sanitized.json	POST /webhook/get-bookings	Moves the caller's upcoming booking to a new time

Both files are sanitized: the Cal.com API key is replaced with YOUR_CAL_API_KEY, and webhook, instance, workflow and version IDs are removed.

Setup
In n8n, go to Workflows → Import from file and import both JSON files.
Add your Cal.com API key. Preferably create a Header Auth credential (name Authorization, value Bearer <your key>) and select it in each HTTP Request node. Otherwise replace YOUR_CAL_API_KEY in these nodes:
Cancellation: find booking, cancel meeting
Rescheduling: HTTP Request, HTTP Request1
Activate both workflows and copy each Production URL from the Webhook node.
In Retell, add two custom functions pointing at those URLs (method POST).
Request bodies

Retell sends function arguments under args.

Cancel

json
{ "args": { "email": "caller@example.com", "reason": "Schedule conflict" } }

Reschedule

json
{ "args": { "email": "caller@example.com", "new_booking_time": "2026-11-20T10:00:00Z" } }

new_booking_time should be an ISO 8601 UTC timestamp.

Flow

Cancellation: Webhook → find upcoming bookings by attendeeEmail → any found? → yes: cancel first booking → reply "canceled" / no: reply "no bookings found, verify email".

Rescheduling: Webhook → find upcoming bookings by attendeeEmail → any found? → yes: reschedule first booking to new_booking_time → reply "rescheduled" / no: reply "no reservation found, check email".

Known limitations
Only the first upcoming booking (data[0]) for an email is cancelled or rescheduled.
The webhooks have no authentication. Add a Header Auth check on each Webhook node and send the secret from Retell's custom function headers.
The success reply is sent without checking the Cal.com response, so a failed API call can still say "canceled" or "rescheduled".
Cal.com API versions are pinned in headers (2026-05-01 for listing, 2026-02-25 for cancel/reschedule). Update them if Cal.com changes the API.
Security

Never commit exports containing a live API key. If a key has been shared or pasted anywhere, rotate it in Cal.com (Settings → Developer → API keys).
