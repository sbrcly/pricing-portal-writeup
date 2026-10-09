# Pricing portal

The internal web app where Etainement's analysts priced inventory against the live market and on-sale managers drew the buy rules the [buyer extension](https://github.com/sbrcly/buyer-extension-writeup) painted onto Ticketmaster. A team system: eight people built it over its life, and I was second by commits. This repository is a write-up of the parts I built; the code is the company's. Case study with drawings: [scottbarclay.dev/work/pricing-portal](https://scottbarclay.dev/work/pricing-portal).

## What it is

A ticket broker holds thousands of tickets across hundreds of events and has to price each one against what everyone else is asking. Analysts work through the events where the company holds inventory, see their own listings next to the current market, and change prices by hand or by rule. Before a big on-sale, managers plan what to buy and draw on the venue map the sections and rows they want.

The portal keeps no data of its own. Everything it shows comes from, and everything it changes goes to, a backend that reads three point-of-sale systems live, keeps its long-term record in BigQuery, and caches what it can in between.

## What I built

- **The rule-authoring map.** During an on-sale a manager needs to say "these sections, these rows, up to this price" in seconds, on a map. Control-click on a seat starts a rule for its section. Control-drag draws a line, and every seat within a band around it is collected by a point-in-polygon test, which sweeps a curved block of seats in one motion. On submit the browser resolves each rule to a list of concrete seat IDs and saves both the readable criteria and the list, so the extension only has to check membership.
- **The analysts' worklist (first generation).** One event in four resizable panels: map, market, own listings, controls. Clicking a section filters the market; hovering a row lights its section. Price edits are staged and saved in one bulk request. A price far enough under the market's lowest comparable opens a blocking popup, editable, before anything is sent.
- **The rules service.** Saving a rule set sanitizes the record, posts a stop alert to buyers if the manager set the stop switch, appends the row, writes per-seat notifications, and warms the cache. The portal reads back through a freshness check; the extension reads the latest row directly.
- **The manual price update and its audit trail.** The backend looks up which of the three point-of-sale systems owns the event and pushes the change. Only if the whole push succeeds does it append an audit row per listing: who, when, old price, new price.
- **Warehouse cost attribution.** Four BigQuery clients, one per product area, and a metadata comment on every query naming the tool and page that issued it, so warehouse cost can be read per page from the job log.
- **The telemetry write buffer.** Logins and page views collected and flushed every ten seconds or every hundred records, with retries and a final flush on shutdown.

## What I would do differently

The role model is enforced only in the browser. Two managers editing one event's rules overwrite each other, last write wins. Five table libraries where one would do. And a test suite before more Sentry.

Stack: React, Node, BigQuery, MySQL, three point-of-sale integrations, Sentry.
