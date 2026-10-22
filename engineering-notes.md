# DAISY — Engineering Notes

DAISY (**Db2 AI Support for You**) is a self-initiated internal project that
applies database and production-support experience to an AI-assisted
investigation workflow. The aim is to turn a DBA question into a clear set of
checks, findings and supporting evidence that another person can review.

The [README](README.md) introduces the capabilities, console and architecture.
These notes explain my contribution, the engineering choices and the limits of
the available validation.

## My contribution and use of AI

I originated the idea and designed the intended flow from a DBA request to
catalog evidence and an investigation handover. My main contributions were:

- **Problem definition:** identifying recurring investigations where selecting
  queries, combining results and preparing evidence created repeated work.
- **Architecture and execution boundaries:** choosing predefined SQL tools,
  with Python controlling queries, calculations and investigation steps while
  AI interprets requests and selects supported operations.
- **Initial database connectivity:** investigating the Python-to-Db2 connection
  approach, learning to use `pypyodbc`, and repeatedly testing until I established
  a connection.
- **Review priorities:** defining the questions an investigation must answer:
  which object was selected, what ran, what failed, and what evidence supports
  the conclusion.

AI assisted development, refactoring, regression-test preparation, review and
documentation. My ownership is the problem, intended behaviour and design
judgement; AI-assisted work still requires understanding and verification.
The recorded automated checks are not a claim that I personally authored or
manually verified every part of the application.

The initial connection experience is my account of integration work. The
offline test results below use simulated connections and do not independently
reproduce that result.

## Design decisions and trade-offs

| Decision | Why it matters | Trade-off |
| --- | --- | --- |
| Terminal workflow with ODBC access | Fits a focused DBA investigation and provides a direct path to catalog evidence. | Operation depends on the native driver, data-source configuration, authentication and permissions. |
| Predefined SQL tools | Keeps permitted queries and inputs explicit, reviewable and testable. | Each new investigation needs deliberate implementation and validation. |
| Deterministic investigation workflows | Runs prescribed steps and calculations consistently, making results easier to compare. | Catalog differences and specialist object types need explicit handling. |
| Optional AI interpretation and tool selection | Makes supported capabilities accessible through natural-language questions. | Adds ambiguity, provider dependence, latency and data-handling considerations. |
| Structured evidence and Excel export | Retains contributing results, warnings and traces for review and handover. | Exported metadata can be sensitive, and separate reads are not automatically a consistent snapshot. |

The central separation is:

**Request interpretation → validated dispatch → predefined tools and workflows
→ catalog evidence → findings and export.**

These are responsibilities, not a guarantee that every request follows an
identical path. Explicit commands, local lookups and AI-assisted requests have
different routes. Workflows can call several catalog tools; glossary and
pasted-output interpretation can complete without a database query.

A saved-query library or an ordinary script is a useful baseline. Much of
DAISY's value comes from organising investigations and retaining evidence.
AI's additional contribution is easier access through natural language, which
needs to be evaluated against the same operations reached through commands.

## Practical challenges and responses

### Establishing a working database connection

The first substantial challenge was finding and testing a Python connection
approach. Useful AI interaction depended on a working data-access path.
Installing a Python package alone did not establish that the native ODBC driver,
data source and authentication would work together.

Connection handling separates access failures from investigation logic, closes
query cursors and marks selected communication failures as an unhealthy
connection. Recorded offline checks cover these behaviours. A missing native
ODBC library prevented a driver-dependent run in the validation environment.

**Lesson:** test the external dependency early and distinguish package setup
from successful end-to-end connectivity.

### Resolving intent while controlling execution

A request such as “check this table” needs both a target and a clear
investigation. DAISY uses explicit identifiers, an active schema, remembered
objects and clarification paths to resolve missing information.

Approved tool registries, argument and identifier validation, SQL keyword
checks, and limits on agent rounds and tool calls constrain execution. Recorded
regression cases include rejecting an unsupported tool and stopping at the
configured limits. Other cases cover remembered objects, active-schema
resolution and malformed routing responses.

**Lesson:** validation must cover the route from the user's request to the
selected operation. Correct individual tools do not guarantee correct routing,
and simulated model replies do not measure live-model accuracy.

### Making incomplete evidence visible

Missing information must change a conclusion. An unresolved object should stop
dependent reads. Absent RTS should remain unknown rather than become a zero
storage value or a healthy status.

Recorded tests check missing-object stops across twelve direct object workflows,
selected missing-location cases, and unknown ratios or freshness when RTS is
absent. A synthetic storage case checks **200 MB of tablespace allocation +
50 MB of index allocation = 250 MB of known allocation**, with contributing
evidence retained.

**Lesson:** show what is known, what is missing and how a result was calculated.
These selected checks do not establish correctness for every catalog variant.

### Preserving the right evidence for handover

The latest visible response may be a glossary answer while the user still wants
the earlier investigation's evidence. DAISY distinguishes the latest response
from retained workflow evidence; a regression test checks that a later question
does not replace the investigation intended for export.

The full result, the sample passed to a model and an exported workbook also have
different purposes. Recorded checks cover full result counts after sampling and
selected workbook contents. Workflow export retains existing evidence, while a
last-result export backed by SQL can rerun its query and return newer or larger
results.

**Lesson:** define which evidence is being summarised or exported. A retained
result, a model summary and a fresh query are different artifacts.

## Validation evidence

Validation focused on investigation behaviour, error handling and the evidence
available for review. The recorded offline checks used controlled examples in
place of live database results and AI responses.

| What was checked | What the recorded checks demonstrated |
| --- | --- |
| Investigation logic and error handling | Selected cases passed for input validation, calculations, missing objects, absent statistics and query failures. |
| AI tool controls | Unsupported tool requests and invalid arguments were rejected in the tested cases. Agent limits stopped further execution while retaining the latest evidence. |
| Operation without optional AI libraries | The covered core workflows passed without those libraries. AI-specific checks were excluded from that configuration and passed in a separate environment with the required libraries installed. |
| Evidence retention and Excel export | Checks confirmed that investigation evidence survived a later unrelated response. Selected result rows, warnings, traces and workbook contents matched across comparison runs. |

Live network connections were blocked during these checks. Workbook tests used
the actual Excel library to verify exported content.

This evidence supports the behaviours exercised by the tests. It does not
establish live Db2 or AI-provider compatibility, model answer accuracy, measured
time savings or operational approval. Those require separate integration checks,
evaluation and human review.

## Control boundaries and remaining work

The exposed operations investigate catalog metadata; they do not offer general
SQL execution or database-changing operations. The design still has important
limits:

- **Database permissions:** SQL keyword checks are a denylist, not a full parser
  or catalog allowlist. The connection layer does not establish database-enforced
  read-only access; appropriate account permissions remain essential.
- **Execution limits:** agent-call limits do not bound every child query or its
  database execution time.
- **Routing and evidence gaps:** AI routes expose different operation sets;
  clarification state lacks expiry; some specialist checks handle missing
  locations inconsistently; model-message truncation can remove evidence.
- **Data handling:** questions, context and sampled catalog evidence can reach
  the configured provider. Logs and exports can retain sensitive metadata;
  general redaction and retention controls are not implemented.

These are concrete priorities for further integration checks and focused
regression tests. Advisory findings continue to require DBA judgement.

## Evaluation and measuring value

The documented benefit is a single workflow for query selection, repeated
checks and evidence assembly. Measured time savings, live routing accuracy,
provider cost and end-to-end latency remain to be established.

A useful evaluation would compare the same approved investigations through:

1. Manual investigation using saved queries.
2. DAISY's explicit commands and deterministic workflows.
3. DAISY's natural-language route to the same capabilities.

| Evaluation area | Evidence to collect |
| --- | --- |
| Correct task selection | Expected and actual operation, target and arguments; appropriate clarification. |
| Supported findings | Conclusions traced to returned evidence and calculations; unsupported claims counted. |
| Handling uncertainty | Responses to missing objects, stale statistics, partial failures and ambiguous requests. |
| Practical efficiency | Time to a reviewed evidence pack, including corrections, reruns and unnecessary tool calls. |
| Operating cost | Model and prompt versions, latency, provider failures and token usage or cost. |

Expected results and completion criteria should be set before measurement, with
both successful and failed attempts included. This is proposed evaluation work;
passing the offline suite is a different form of evidence.

## Engineering lessons

- **Test integration risks early.** A working connection is a foundation for
  useful investigation features.
- **Give AI a defined responsibility.** Flexible interpretation can coexist with
  explicit operations, validation and deterministic calculations.
- **Treat evidence as part of the product.** Findings need supporting results,
  visible uncertainty and a trace another DBA can follow.
- **Separate software verification from AI evaluation.** Simulated tool tests
  and real-model assessments answer different questions.
