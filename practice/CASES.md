# Cases

Real choices. Newest at the top.
Each case opens with the rule and closes with the incident.

`Taken` = method.
`Imposed` = cost of not being able to hold the method.
`Imposed` only appears when decision and outcome diverge.

If the code can express the fix, the diff is a patch and it looks like the file.
If the structure cannot express what is asked, the structure changes.

---

## Video rendering that the shared host would not allow

**Rule:** A fixed price that hides a hard limit is not cheaper. It is a deferred cost with interest.

- Asked: at HivisionLED. Deploy the video rendering service on the shared host at ONE.com. The function used memory within the allowed margin by the numbers. The host blocked it anyway.
- Taken: contacted support. Session with an engineer. Asked for a memory increase. Denied. Tested a cheaper alternative the boss recommended. Same ceiling. Chose App Engine. Pay-per-use. Scalable. Accepted that the bill would move with demand.
- Rejected: raising memory on ONE.com — host said no. The cheaper host — same ceiling. Limiting file size — clients had already threatened to leave. Rendering on the client — tried, did not work. A VPS (OVH, DigitalOcean) — the maintenance it adds is work App Engine already covers.
- What broke otherwise: the service had already suffered outages and IP blocks on the old host. Clients were threatening to leave if file size was limited. 4K was arriving. Without a scalable host, the service would have stopped serving its own customers. A provider change was already cooking regardless of the choice.
- Number: ONE.com fixed at 9.99 USD, hiding the ceiling. App Engine 12 USD normal, up to 20 USD extreme. Variable cost bought scalability the fixed price could not.
- After: App Engine held. Let us grow the client base and add planned service types that were blocked by the old ceiling.

---

## Angular hands, React project, week one

**Rule:** When the stack changes without warning, the codebase is the only documentation. Read it, copy the pattern, do not inject your style, ship in the first week.

- Asked: at MetLife. Applied for an Angular role. Was told React Native. Said I did not know React Native. No option given. On day one, the stack was ReactJS, an old version. First sprint, 15 days, deadlines already running.
- Taken: learned on the move. No docs, so read the code and replicated the pattern. Every new screen was a reason to study another flow. Shipped features within the first week.
- Rejected: quitting — never an option. Being moved to another project — there was none. Injecting my own style — would have complicated things for the team and hidden the gap. Waiting to master the old version before shipping — the sprint would not have waited.
- What broke otherwise: if I missed the sprint, the whole team went to bench. The boss had staked the team on one person who was not there. The two experts assigned never showed — one on leave, the second never connected. The stakes were the team's, not mine.
- Number: 15-day sprint. Features shipped within week one.

---

## Wearables in Cordova, with no plugins allowed

**Rule:** When the restriction makes the request impossible, the finding is the deliverable — not a POC dressed as one.

- Asked: at Hexalud. Add wearables to the app through Cordova, without using plugins. The team insisted it was possible.
- Taken: no. Explained why — required native communication that Cordova without plugins cannot do. Every path that did not contradict the no-plugins rule required native communication. The no was a finding, not an opinion.
- Imposed: months of investigation demanded on an already-answered call, to prove the no. Went through API permissions, library wrappers, every path. Nothing contradicted the finding. The months were the cost of not being believed, not the procedure.
- Rejected: any wrapper or library that would have done the job — the no-plugins rule forbade it. Bending the rule without saying so. Shipping something that looked like it worked. Staying silent while months were spent on something already known to be impossible.
- What broke otherwise: shipping a Cordova wearable without plugins would have failed in production. No middle path. Anything that looked like a fix was a wrapper, which violated the rule.
- Number: months of investigation on an already-answered call.
- After: a POC with React Native was authorized. Presented as a full success. The stack changed, not the requirement.

---

## A remote print job inside a closed POS network

**Rule:** A no that only one person stands behind is not a decision. A hack that works one week buys trust it cannot pay for.

- Asked: at El Cometa. The client wanted to print tickets remotely from their web app. Their POS provider ran a closed network and would not grant access. Extra request on top of an online sales service the client already paid for. Client did not want to pay extra.
- Taken: no. Explained why: no access, no integration. Proposed a dedicated mini PC. Proposed a Raspberry. Both ignored.
- Imposed: someone else accepted the request anyway. The hack was built. Synced Firestore with the printer. The hack ran for a week. A network latency error duplicated tickets. Investigation confirmed the hack was not adequate. Client ended up buying dedicated equipment.
- Rejected: going to the POS provider to demand access — policy forbade it, the POS would only integrate if we covered the cost. Saying no to the client flat out — the boss would not carry that either. Rejecting the project entirely.
- What broke otherwise: the hack worked for a week, worse than not working. It bought trust it could not keep. When latency hit, tickets duplicated. Four reputations paid: the client's, for exposing providers outside their POS; my boss's, for refusing to absorb cheap hardware and for a no with nothing behind it; mine, for looking like the one who does not contribute; the team's, for shipping a risky hack as a solution. The mini PC would have cost less than the week.
- Number: 1 week of false success. 1 duplicated-ticket incident.

---

## A domain migration and a file nobody versioned

**Rule:** When a migration breaks something that was never versioned, the deliverable is not the fix. It is the audit, the versioning, and the README that would have prevented it.

- Asked: at HivisionLED. New domain purchased. Migrate the app to it. The boss had already redirected clients to the new address and closed the old domain. The code was untouched.
- Taken: file-by-file comparison against the source server. Tried to revert. Server denied it, read-only. Traced a silently failing call through network logs to an unversioned JSON file that told the app where to fetch its data. Added the file. The app came back.
- Rejected: asking the boss to reopen the old domain — investigation showed nothing wrong in the code. Assuming the documented migration was complete — that is what had just failed. Escalating to infra before auditing the source server.
- What broke otherwise: without the audit, the app stayed down. Clients had no way in. The boss had already closed the old domain, no fallback. The only path was forward, and forward meant finding a file no one knew existed.
- Number: 3 extra hours.
- After: config files are versioned. Every project gets a README documenting its setup.

---

## 600 modules for one URL

**Rule:** A change that has to be repeated 600 times is not local. Move it to the layer that owns it, tag it, scope it, and give it one rollback switch.

- Asked: at Banamex. Migrate endpoints in an Android app. The project lead asked to enter each module and add a redirect branch for the URL. Around 600 modules.
- Taken: network layer. An interceptor. Fires only with a Kotlin reflection tag. Concurrent cache minimizes memory leaks. One single declaration at the network layer makes the redirect visible. Rolled out by groups of 10 endpoints.
- Rejected: the lead's approach — 600 modules for one URL. Global interceptor without a tag — hides the redirect behind a layer with no visible owner. Doing it at the repository level — repeats logic that belongs in the use case. Touching each module to keep the change "local" — not local once it is 600 of them.
- What broke otherwise: 600 diffs, 600 chances to get the tag wrong, 600 places to remove once the migration ends. Each one a chance to break a banking app in production. The lead's concern about risk was correct. His method multiplied it.
- Fallback: both. Flag first, error handler second. If the flag is on, the interceptor applies. If the network call fails, fallback to original behavior. No silent failure. No half-migrated state.
- Rollback: one single flag for the entire migration. Turning it off returns the app to its original behavior everywhere at once.
- Number: 600 modules avoided.

---

## Static XML that could not become dynamic

**Rule:** If the structure cannot express what is being asked, change the structure. A refactor for expressiveness is not a rewrite for taste.

- Asked: at BrightSign. Add multiple presentation types to an app that generated presentations from XML. Built on a public AdminLTE template. Only produced one type. The new requirement needed several.
- Taken: aggressive refactor. Removed the template's boilerplate. Moved document creation options to the server. XML portions stopped being static and became data the server decides.
- Rejected: extending the existing code — XML fragments were hardcoded, one per type, adding types meant multiplying static blocks. Keeping the template as-is and layering on top — the template was the source of the mess.
- What broke otherwise: a wrong move would have taken the whole app down in production. The refactor was not the safe option. It was the only option that made the requirement possible.

---

## 60,000 paid licenses written wrong

**Rule:** Before touching data, audit. The audit defines the target set. The transaction defines the safety. The batch is only the mechanism.

- Asked: at Injectronic. Portal generates the update key that unlocks automotive scanning devices. An update produced wrong keys for paid licenses. Around 60,000 affected. Window to fix was short.
- Taken: audit script first, to identify only the IDs damaged by that update. Then a chain of Firestore batches with transactions. All or nothing. Each batch committed only if the previous succeeded. Then the key generation fix deployed.
- Rejected: fixing by hand — window too small. A batch over the whole collection — a wrong filter would have overwritten good licenses. Writing the batch before the audit — the audit is what made the target set safe.
- What broke otherwise: a hand fix would not finish in time. A full-collection batch would have destroyed valid licenses; Firestore gives no easy rollback. A batch without the audit would have mixed broken keys with working ones. A partial batch without transactions would have left corrupted licenses mid-fix.
- Number: 60,000 licenses.

---

## A discontinued key library inside a no-new-dependencies rule

**Rule:** When a rule forbids new dependencies, wrap what already works. Do not skip the native side to make it smaller.

- Asked: at MetLife, React Native. Generate a cryptographic key, store it in the phone's secure vault. The dependency doing this was discontinued and broken. No new libraries allowed. The app could not start without a unique secure key per installation.
- Taken: wrapped a working JS version of the same functionality. Bridged to Kotlin and Swift for the native side. Only key generation was needed, so the surface was small.
- Rejected: adding another library, even though one existed. Waiting for the discontinued library to be patched. Generating the key in JS only, without the native secure vault.
- What broke otherwise: the app would not start. A JS-only key would not be in the secure vault, breaking the security requirement. Adding a library would have violated the rule that existed for a reason.

---

## A Kong plugin no one else would maintain

**Rule:** A paid dependency without support is not cheaper. The cost of learning the tool is the price of not depending on a dead vendor.

- Asked: take over Kong after a colleague left. Make it the DNS replacement: dynamic subroutes to different services, one gateway, stop creating DNS records dynamically in AWS. Also required a Kong plugin translating HTTP calls into Kafka. No official plugin existed.
- Taken: learned Lua and Kafka. Wrote the plugin from scratch. Weeks of testing. Wrote implementation documentation for Kong. The plugin was owned by the team, not a vendor.
- Rejected: the paid plugins — support had stopped, and the library policy was strict. Keeping dynamic DNS at 1200 USD. Adopting a community plugin without official support.
- What broke otherwise: an unmaintained plugin would have shipped under a company-level policy that forbids it. If it broke later, no vendor, no patch, no one to escalate. The strict library policy existed precisely for that. Approval alone would not have saved it.
- Number: 1200 USD/month DNS cost. 60% reduction. Kong adopted company-wide.

---

## A migration that outgrew every storage engine it was supposed to use

**Rule:** When no engine fits the whole payload, split the job by what each engine is good at, not by what the catalog wants.

- Asked: migrate IBM Worklight with IBM JSON Store to React Native. Estimated at 6 months. Offline mode mandatory. JSON documents up to 10 MB. Catalog near 2 GB. Fully offline app.
- Taken: a hybrid. MMKV for metadata that includes only what is needed to render plus the file address. Real payload encrypted and stored on disk. Only the requested file is decrypted and loaded.
- Rejected: WatermelonDB, SQLite, any SQL-backed option, even with a custom wrapper. Staying on MMKV alone. Dropping offline or shrinking the catalog to make storage fit.
- What broke otherwise: SQL engines threw "document too long", overflowed memory, or closed the app on save. The catalog could not be split without breaking offline. MMKV alone collapsed on performance once the payload grew. Without the hybrid, the app could not ship.
- Number: 5 million MXN at risk. 6 months estimated, 9 delivered. 3 extra months on POCs.

## An .env pushed while updating .gitignore

**Rule:** A secret in a diff is a stop, not an urgency question. Revoke the leaked credential. Add a pipeline check so the next one never lands. Human review is not the control.

- Asked: updating `.gitignore`, the line protecting `.env` was removed by accident. The `.env` was committed on the next push. The reviewer did not review. A credential hack followed. Services were limited, so the attack attempts only caused brief interruptions.
- Taken: fixed in the same shift, 4 hours from leak to fix. Revoked the leaked credential. Added a pipeline check for secrets. Did not trust the next review to catch it.
- Rejected: leaving the credential alive because the window was short. Also rejected: revoking and then trusting the reviewer. Revocation kills this leak. The pipeline stops the next one. Neither replaces the other.
- What broke otherwise: the service stopped responding for seconds. The credentials were live for 4 hours. Without revocation, the leaked key kept working after the file was gone from the branch. Without the pipeline check, the next leak would have been identical.
- Number: 4 hours from leak to fix. 1 credential revoked. 1 pipeline check added.

---

## Ping One Identity added to a nearly-done sprint

**Rule:** An added scope inside a running sprint is a documented risk, not a negotiation. Document, accept, deliver. Do not pretend it fits.

- Asked: at MetLife Seguros. Migration to React Native. The estimate covered structure and dashboard only. Near the end, the request added Ping One Identity. iOS was the hardest part: URL redirect trust.
- Taken: evaluated. Accepted with documented risk and possible delays. Delivered on time.
- Rejected: refusing the identity work because the estimate had already closed. Also rejected: absorbing it in silence, with no written risk, so a slip would look like a missed promise.
- Imposed: a teammate did not agree and did not do the development. The team fractured over the decision. That was the cost of accepting, not a reason to drop the client's requirement.
- What broke otherwise: without the added scope, the migration would have shipped without identity, which was the reason for the change. Without the written risk, a delay would have had no record. It would have looked like a missed estimate.
- Number: delivered on time. 1 teammate off the work. The written risk is the contract for the next scope added mid-sprint.

---

## A stale type file took the UI down for a day

**Rule:** Type protection is only as current as the file that defines it. A protection you forgot to update is not a protection. When you know the cause, rollback is not the first move.

- Asked: at Cibeles. Deployed a catalog update. The release broke the types in the UI. The interface did not render. Detected next day, after 4 user reports. The app had remote type protection, but the file had not been updated.
- Taken: fixed immediately by editing the types file. Added a development validation to auto-generate the types going forward.
- Rejected: rolling back. The cause was known on sight. A rollback would have delayed the catalog and hidden the real gap: the stale file, not the release. Also rejected: investigating in hot with the UI down for users.
- What broke otherwise: the UI was down for a day. 4 users reported before detection. The type protection existed. The file was stale. The fix was one file edit. The delay was a day.
- Number: 4 user reports. 1 day of broken UI. 1 file edit to fix. 1 dev validation added.

---

## A schema change on a project already in production

**Rule:** Version the API before the schema changes, not after. Once clients are in the wild, dual-schema acceptance is a fix, not a design.

- Asked: at Industrial Sources. A new field had to be added for the date. The previous field was deprecated because it was not ISO format. Clients were already in production on the old field.
- Taken: dual-schema acceptance as an emergency fix so clients did not break. Then versioning added to the API so the change could land cleanly.
- Rejected: forcing an update on the client. The clients were live and had not migrated. Breaking them was not the price of the new field.
- What broke otherwise: without dual-schema acceptance, every client on the old field would have broken the moment the new schema shipped. Without versioning, the next schema change would have repeated the same emergency. The dual schema bought time. The versioning removed the need for it.
- Number: 1 emergency dual-schema fix. 1 versioning layer added. 1 deprecation resolved.