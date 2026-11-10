# Bhoomi Setu: A Plain-English Project Guide

## 1. What this is and who it is for

Bhoomi Setu is a prototype (an early, experimental version) of a tool that helps a person understand their land records before they take action. It collects the documents about one piece of land, pulls out the key facts, compares them using fixed rules, and then suggests practical next steps. Everything in it uses made-up demo data. It is not a government portal, it does not give legal advice, and it never decides who owns land or what is legally correct.

The material describes the intended user as a citizen who has to prepare for a land survey. Such a person may need to piece together old records, family or inheritance details, survey paperwork (the "Parcha"), and map information. The hard part, according to the README, is knowing what evidence exists, what agrees, what needs a second look, and what to prepare next.

The project was built by a team called "The White Knight" and, from the wording of the material, was prepared for a hackathon (a timed competition for building projects). It extends an earlier prototype by the same team. The commit history of that earlier work is kept on purpose to show how the project developed over time.

## 2. The big picture: the main parts

The material groups the code into many small areas. For a newcomer, it is easier to think of nine main parts. The folder names below come from the file notes provided.

**1. The web pages (what the user sees)**
Home page, case dashboard, documents, verification, parcel view, survey, timeline, family, next action, review packet, create-case form, and a public verification page. These live in `src/app` (with the case pages under `src/app/cases`, the form under `src/app/create-case`, and the public checker under `src/app/verify`).

**2. Shared screen pieces**
Reusable building blocks: the map panel, the folding survey map, the satellite comparison panel, the language switcher (English and Hindi), the page frame with navigation, and decorative scenes such as a rotating globe and an animated village. These live in `src/components`.

**3. The web address layer (the API)**
An API is a set of addresses the screens use to ask the server for information or to send it something. These addresses handle cases, documents, verification, parcel information, official records, earth observation, review packets, and signed records. They live in `src/app/api`. The list is documented in `docs/API.md`.

**4. Case and document handling**
Creates and loads cases, supplies the demo documents, and builds a timeline of what has happened. The files are mostly in `src/server` (for example `case-state-service.ts` and `document-application-service.ts`), with some client-side helpers in `src/services`.

**5. Reading documents (extraction)**
Turns document text into structured facts such as khata number, area and name. There is a simple rule-based reader in `src/services/document-processing.ts` and an optional AI-based reader in `src/server/extraction.ts`, guided by instructions in `src/server/extraction-prompt.ts`.

**6. Verification (the rule-based checking)**
Compares areas and family details across documents and records a result for each check along with where the evidence came from. The main file is `src/server/verification-service.ts`. The rules are explained in `docs/VERIFICATION_ENGINE.md`.

**7. Parcel intelligence and maps**
A parcel is a single plot of land. This part stores the plot's boundary shape, checks it is valid, calculates its area, and compares area figures from different sources. Files are in `src/server`: `geospatial-service.ts`, `parcel-geometry-service.ts`, `parcel-area-comparison-service.ts`, and `parcel-intelligence-service.ts`.

**8. Context from outside the case**
Two features add background information without changing the main results:
- Official-style records: a stand-in for government records, searched and imported per case (`official-record-service.ts`, `government-adapter.ts`).
- Earth observation: fixed, made-up satellite-style snapshots compared to describe change such as vegetation or buildings (`earth-observation-service.ts`, `imagery-provider.ts`).

**9. Guidance, review packets, and trust features**
- Guidance turns results into ordered, plain-language next steps (`guidance-service.ts`).
- Review packets are drafts a user can prepare about a flagged issue. Once marked ready they are locked, and nothing is sent to any authority (`review-packet-service.ts`).
- The trust layer lets a verification result be "anchored" (given a tamper-evident fingerprint) and later checked by anyone on a public page. The type definitions are in `src/types/anchor.ts`; the exact code files for anchoring are not named in the file notes (see Open Questions).

**Supporting areas**
- Storage: `src/server/database.ts` (local database or hosted database).
- Safety checks for demo-only input: `src/server/prototype-mode.ts`.
- Usage timing recorder: `src/server/metrics.ts`.
- Test benchmark: `src/evaluation`.
- Documentation: `docs`.
- Automated checks on every code change: `.github/workflows/ci.yml`.

## 3. How the pieces work together: one typical journey

The README recommends a guided path using the main demo case, called `demo-family-001`. The screens go in this order: Dashboard, Documents, Verification, Parcel Intelligence, Official Records, Earth Observation, Next Action. Here is the journey in plain words.

**Step 1: Open the case.** The user lands on the dashboard. It shows the case's progress, the differences that need attention, a map, the evidence, and links to the detail pages. The demo case describes a fictional family plot in "Demo District" with khata (land account) number DEMO-128 and khesra (plot) number DEMO-456.

**Step 2: Look at the documents.** The user attaches approved demo documents to the case. The app only accepts fixed sample documents from a list. Arbitrary uploads are rejected. Each document is shown with its text, how well it could be read, and where its facts came from.

**Step 3: Extract the facts.** The app reads labelled lines from the text to find things like identifiers, areas and names. Optionally, an AI service can suggest facts as well. Either way, every fact is tied back to the part of the text it came from.

**Step 4: Verify.** Ordinary program code (not an AI) compares the facts. For the main demo case, the old document says the area is 1.20 acre, the survey says 1.02 acre, and the mapped boundary works out to about 1.0243 acre. The result is:

| Comparison | Difference | Result |
| --- | ---: | --- |
| Historical vs Survey | 15.0% | Potential issue |
| Historical vs Mapped boundary | about 14.64% | Potential issue |
| Survey vs Mapped boundary | about 0.42% | Consistent |

In plain terms, the old record disagrees with the other two, while the other two roughly agree. The app reports this as a difference to look at. It does not say any record is right or wrong.

**Step 5: Check the parcel view.** The parcel panel shows the plot's identity, its map outline, the calculated area, and how each source compares.

**Step 6: See the extra context.** The Official Records page shows imported stand-in records, labelled as not authoritative. The Earth Observation page shows fixed sample satellite-style comparisons, labelled as not legal or boundary evidence. Neither changes the verification result.

**Step 7: Get next steps.** The guidance page lists ordered steps, checklists and cautions. The user can then create a review packet for the flagged issue, add notes, and mark it ready. Nothing is submitted anywhere.

**Optional step: Anchor the result.** On the verification page the user can anchor the result, which records a fingerprint of it. Anyone can later use the public verify page, with no login, to check whether the record still matches.

**The control case.** A second case, `demo-family-002` (DEMO-902 / DEMO-114), is the "everything agrees" example. Its historical and survey areas are both 1.25 acre and the mapped area is about 1.2514 acre. Area verification passes, and the family check reports "insufficient evidence" rather than inventing a problem.

## 4. Where the data lives and how it gets in

**What is stored.** According to the file notes for `DATA_MODEL.md` and `PERSISTENCE.md`, the app stores cases, documents, verification results, guidance, review packets, anchors and audit records (a log of what happened). Parcel boundary shapes are stored per case. Imported official-style records are saved per case. The README also notes that case data is kept separate so one case cannot read another's.

**Where it is stored.** There are two options, per `database.ts` and its tests:
- A local database file (SQLite, a small database that lives in a single file) is used by default.
- A hosted database (Postgres, a widely used server database, with Supabase named in the project notes) is used when it is configured.
- On the hosting platform Vercel, the app is designed to fail safely if no database has been set up.

The audit log is set up to be append-only, meaning entries can be added but not edited or deleted.

There is also a simple in-memory store (`src/services/mock-case-repository.ts`) holding two demo cases. The notes call it a stand-in for a real database.

**How data gets in.** The material shows four routes:
1. Demo cases come pre-loaded as fixed sample data.
2. A user can create a new case with a form, which has a one-click sample fill. The app checks that the inputs look like demo identifiers and labels, not real personal data (`prototype-mode.ts`).
3. Documents are attached from an approved list of fixture documents, never from free uploads.
4. Official-style records are searched and imported from fixed synthetic records.

A "reset" button returns a demo case to its original data.

The documents say clearly that only synthetic information should be put into this prototype. `PRODUCTION_ARCHITECTURE.md` separates what is built now from a future production design, including security and upload needs, so real uploads are a future matter.

## 5. The AI parts

AI plays a small and fenced-in role.

**What it does.** An optional AI service reads the text of a synthetic document and suggests structured facts, such as identifiers and areas. The README says it can use Gemini or OpenAI for this. It is optional, and the simpler rule-based reader exists as well.

**How it is kept in check.**
- The instructions given to the AI are versioned and tell it to treat the document as untrusted data and to extract only facts that are explicitly stated (`extraction-prompt.ts`).
- Each fact must follow a shared format, cite the part of the text it came from, and pass a "grounding" check, meaning the fact has to actually be supported by that cited text. Facts that fail are not accepted, and failures are recorded safely.
- Extracted facts are stored as candidate or accepted facts, per `DOCUMENT_PIPELINE.md`.

**What it does not do.** The README is explicit: the comparison and the pass or issue verdicts are done by ordinary code, not by an AI. The AI is not used to decide whether records match, and it is not used to give legal conclusions.

The material also describes language support (English and Hindi) for labels, guidance and survey stages. It does not say how the Hindi text is produced, so I have not assumed it comes from AI. Changing language is tested to confirm it does not rerun verification.

## 6. Glossary

- **Prototype:** An early working version built to show an idea, not a finished product.
- **Synthetic data:** Made-up data that looks realistic but describes no real person or land.
- **Case:** One person's or family's bundle of land records inside the app.
- **Parcel:** A single plot of land.
- **Khata:** A land account number that groups a holder's land records. The material uses it as an identifier.
- **Khesra:** A plot number used to identify a specific piece of land. The material uses it as an identifier.
- **Mauza:** A revenue village or area unit. It appears in the case details. The material does not explain it further.
- **Circle and District:** Administrative areas used to locate a case.
- **Parcha (Khanapuri Parcha):** A survey record document. The material uses the term without a full definition.
- **Acre:** A unit of land area. The app converts other units to acres to compare them.
- **Geometry / boundary:** The drawn outline of a parcel on a map.
- **Extraction:** Pulling key facts out of a document's text.
- **LLM:** A "large language model," the kind of AI that powers chat assistants such as Gemini or OpenAI's models.
- **Deterministic:** Always giving the same answer from the same input, because it follows fixed rules with no guesswork.
- **Verification:** Comparing facts across documents and recording the outcome.
- **PASS / POTENTIAL_ISSUE / INSUFFICIENT_EVIDENCE:** The three possible verification results: things agree, things differ and need review, or there is not enough information.
- **Traceability / evidence reference:** A link from each fact or result back to the source it came from.
- **Grounding:** Checking that an AI-suggested fact is truly supported by the text it cites.
- **Review packet:** A draft set of notes about a flagged issue, which locks once marked ready. It is never sent anywhere.
- **Anchor:** A tamper-evident fingerprint (hash, a short code computed from content) saved for a verification result.
- **Earth observation:** Satellite-style imagery used for background context. Here it is fixed sample data.
- **Official record adapter:** A stand-in for connecting to government record systems. Here it returns demo-only data and cannot submit anything.
- **API:** The set of server addresses the screens use to fetch or send information.
- **Audit log:** A history of actions, set up here so entries cannot be changed.
- **CI:** "Continuous integration," automatic checks that run each time code changes.

## 7. Where to start reading, and what to be careful with

**Suggested reading order**
1. `README.md`, for the idea, the hero case and the numbers.
2. `docs/PRODUCT.md`, for the purpose, user journey and non-goals.
3. `docs/SAFETY.md`, for the limits and release checklist.
4. `docs/DEMO_SCRIPT.md`, for the click-through and the wording to use.
5. `docs/ARCHITECTURE.md` and `docs/DATA_MODEL.md`, to see how the parts connect.
6. `docs/VERIFICATION_ENGINE.md` and `docs/DOCUMENT_PIPELINE.md`, for the checking rules and AI handling.
7. `docs/SYNTHETIC_DATA.md`, to understand what each demo case is meant to show.

To try it, the README says to open the public demo at bhoomi-setu-xi.vercel.app, or run `npm run dev` and visit `http://localhost:3000`. Begin with `demo-family-001`.

For code, the biggest hub is the verification service, and the shared language helper is used almost everywhere. Those are good places to see how things are wired.

**What to be careful with**
- **Do not call it an official service.** The project labels itself an independent prototype, with synthetic data only, and not legal advice or a government portal.
- **Do not describe results as legal conclusions.** A "potential issue" means two sources differ. It does not say who owns the land, whether a title is valid, or what an inheritance outcome should be.
- **Use only fake data.** The app is built to reject inputs that look like real personal details. Real land or personal records should not go in.
- **Treat the extra context as background.** Official-style records and satellite-style imagery are labelled as non-authoritative and are kept from changing the core results.
- **Do not let AI make the verdict.** Keep the split: AI may suggest facts, code does the comparing.
- **Watch the wording.** The demo script has wording rules to keep claims accurate. Follow them when presenting.
- **Some documents are empty.** In `docs/architecture-research`, several files (`COPY.md`, `DEMO.md`, `PROGRESS.md`, `QUALITY.md`, `SECURITY.md`, `SPATIAL.md`) are placeholders with no content. Do not rely on them.
- **Documents may be out of date.** The automatic code map was built from a specific commit (`ec298af7`) and may be stale. The "Phase 1.5 audit" is a review of an early version and may not reflect the current code.
- **The README was cut off** in the material provided, so the full list of capabilities and later sections (such as future scope and checks) was not available.

## 8. Open questions

These are things the material did not make clear.

1. **Who actually uses it, and is anyone using it?** The README describes a citizen as the intended user, but nothing says whether real users have tried it. The project is described as a demo with synthetic data.
2. **What exactly does "anchoring" do?** The README and API notes mention anchoring, a signed result, and a public check page. The material does not say where fingerprints are stored beyond the database, whether any outside service is involved, or what "signed" means in practice. The names of the code files that do the anchoring are also not listed.
3. **Which AI service is used by default?** Gemini and OpenAI are both mentioned as options. It is not stated which is used in the public demo, or whether the demo uses AI at all.
4. **How is Hindi text produced?** The app supports English and Hindi, but the material does not say who wrote the Hindi text or whether it was reviewed.
5. **How are the thresholds chosen?** The comparison rules label differences as consistent, review, or potential issue, based on "demo thresholds." The exact cutoff values are not given, and neither is the reason for choosing them.
6. **What is the "village simulation"?** The code map shows a large cluster of code for an animated village scene. It appears to be decoration, but the material does not confirm its purpose.
7. **Which database does the public demo use?** Both local and hosted options exist. The material does not say which one runs the public demo.
8. **What do the benchmark results show?** An evaluation benchmark and `EVALUATION.md` exist, but no actual results or scores are included.
9. **What is the relationship to the earlier project?** The README says Bhoomi Setu extends an earlier prototype that appears to carry the same name, linked to a repository called `bhoomi-check`. The material does not explain what changed between the two.
10. **What is the future path?** `PRODUCTION_ARCHITECTURE.md` and `OFFICIAL_RECORD_ADAPTER.md` mention future lawful integration and a production design, but the material gives no timeline, plan, or partner.
11. **Several terms are not fully defined.** Words like Mauza, Circle and Parcha are used without explanation. Their exact meanings in this context should be confirmed with the team.
12. **What does the `.21st` folder do?** It holds design settings, but its summary file is mostly unfilled, so its role is unclear.