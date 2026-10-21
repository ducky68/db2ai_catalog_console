# DAISY — Engineering Notes

*Updated 22 October 2026. Scope: the modular Db2 catalog investigation application.*

DAISY stands for **Db2 AI Support for You**. The project began with a practical
DBA problem: a useful investigation often needs several catalog queries, careful
interpretation and a clear evidence trail for the next person.

My aim was to bring those steps into one understandable workflow. My
database and production-support experience shaped the priorities: know what will
run, recognise incomplete evidence, make failures visible and leave results that
another person can review.

These notes distinguish my account of the original idea and connection work,
design rationale supported by the implementation, and results recorded by
automated checks. They are an engineering account, not a complete development
diary. The [README](README.md) introduces the capabilities and architecture.

## My contribution and AI assistance

### What I originated and designed

The project idea and intended end-to-end flow were mine. I identified the DBA
investigations I wanted to simplify and worked out how an assistant should
interact with the database.

A central decision was to put SQL inside predefined Python tools. The model can
select an available operation and provide arguments; the application retains
control of query construction, calculations and workflow steps. This made the
execution path something I could reason about using familiar production-support
questions: what will run, what could fail and what evidence will remain?

I also worked through the initial Python-to-Db2 connectivity problem. My account
of that work is a couple of weeks of investigation and repeated testing,
including discovering and learning to use `pypyodbc`, before establishing a
connection. The project needed that working data-access path before its
higher-level features could be useful.

That earlier connection result is my reported integration experience. The saved
offline validation does not reproduce it or independently establish acceptance
in an operational environment.

### How AI contributed

AI assistance was used during development, source inspection, refactoring,
regression-test preparation, review and documentation. These notes were also
drafted with AI assistance, using my existing contribution account, the local
implementation and saved validation records.

The records do not establish a reliable percentage of code written by me versus
AI. I do not claim sole manual authorship or personal line-by-line review of the
entire application.

| Area | My contribution or the recorded activity | What the evidence establishes |
| --- | --- | --- |
| Problem and scope | I originated the idea and selected the intended investigation flow. | My account of the project; source code alone cannot establish authorship. |
| Architecture | I chose predefined SQL tools and application-controlled execution. | My design account, consistent with the inspected implementation. |
| Initial connectivity | I investigated and repeatedly tested the Python connection approach until it worked. | My account of earlier integration work; the later offline checks use simulated connections. |
| Implementation and refactoring | AI assisted with code development, inspection and package reorganisation. | Recorded changes and checks support particular implementation claims, without assigning an authorship percentage. |
| Testing and review | AI assisted with regression cases and automated review; I retain responsibility for understanding and accepting the work. | Saved test results cover selected behaviour. Automated review is not independent human sign-off. |

### What I review and what has been verified

My review priorities are the selected object and investigation, the permitted
operation, query inputs, calculations, stopping conditions and the evidence
behind a conclusion. These are the questions that follow from my design;
they are not a claim that every execution path has received personal manual
verification.

The contribution I can describe personally is the problem definition, design
choices and earlier connection testing. The software checks described below are
recorded automated verification. Live model quality, every catalog variant and
measured productivity improvement remain separate evaluation work.

## Why this design

| Decision | Reason | Trade-off |
| --- | --- | --- |
| Terminal application with ODBC access | Fits a focused DBA investigation workflow and provides a direct path to catalog evidence. | Deployment still depends on the native driver, data-source setup, authentication and permissions. |
| Predefined Python tools | Keeps SQL and permitted operations inspectable and testable. | A new investigation requires implementation and validation of another operation. |
| Deterministic workflows | Runs prescribed steps and calculations in a known order. | Fixed logic can contain incorrect assumptions and needs explicit handling for catalog variations. |
| Optional natural-language routing | Helps users reach existing capabilities without remembering every command. | Introduces ambiguity, provider dependence, latency and data-handling considerations. |
| Structured evidence and Excel export | Keeps results, warnings and traces available for review and handover. | Exported metadata can be sensitive, and separate reads are not automatically one consistent snapshot. |
| Separate routing, data-access, workflow and output modules | Makes responsibilities easier to locate and review. | Moving files alone does not reduce the complexity of large methods or remove existing defects. |

The core division of responsibility is:

**Request interpretation → validated dispatch → predefined tools and workflows →
catalog evidence → findings and export.**

This describes responsibilities. Commands, local handlers, the structured router
and the optional tool-calling agent follow different paths. Workflows can invoke
several catalog tools, while reference lookups can complete locally.

A saved-query library or a deterministic script is a meaningful baseline. Much
of DAISY's value comes from organising investigation steps and evidence. AI adds
a natural-language entry point; it still needs to demonstrate an advantage over
using the same operations through explicit commands.

## Challenges and design responses

### Establishing the database connection

The first substantial challenge in my account was finding a working
Python-to-Db2 connection approach. A successful package installation did not
establish that the native driver, data-source configuration and authentication
would work together.

The implementation separates connection handling from investigation logic. It
closes query cursors and marks selected communication failures as an unhealthy
connection. A later local validation attempt found that the native ODBC library
was unavailable on that host. That is a separate environment finding; it does
not establish the cause of my earlier difficulties.

**What I learned:** test the external dependency early and distinguish Python
package installation from successful end-to-end connectivity.

### Giving the model useful access within a controlled tool set

Natural-language requests are open-ended, but database execution needs clear
boundaries. I chose predefined tools so that the model selects an operation
whose query and expected inputs can be inspected.

The native agent checks tool names against its registered set. Tool schemas,
identifier validation and SQL keyword checks add controls at other points.
Saved regression cases include rejecting an unsupported tool and stopping at
configured agent-call limits.

The limits matter. A keyword check is not a complete SQL parser, and the
connection layer does not establish database-enforced read-only access.
Permissions must also be set appropriately. A prompt telling the model what it
may do does not replace application enforcement.

**What I learned:** put execution authority in code and describe each control
according to what it actually checks.

### Resolving ambiguous questions and follow-up context

A request such as “check this table” needs both a target and an intended
investigation. DAISY uses explicit identifiers, an active schema, remembered
objects and clarification paths.

Routing order matters: two individually correct handlers can produce the wrong
overall behaviour if the earlier one captures a request intended for the later
one. AI-enabled and AI-disabled routes also differ, and the structured router
and native agent expose different operation sets.

Saved tests cover selected object references, active-schema resolution and
malformed route JSON. They do not measure how reliably a live model chooses the
right investigation. Clarification state also lacks an expiry check.

**What I learned:** test the path from the user's request to the selected
operation, including missing information and follow-up state.

### Turning incomplete metadata into defensible findings

Missing evidence should affect the conclusion. An unresolved object should stop
dependent reads; absent RTS should not become proof of zero storage or current
statistics. Catalog dependency information also has a narrower scope than a
complete application impact assessment.

The saved tests check missing-object stops across twelve direct object
workflows, selected missing-location cases and unknown ratios or freshness when
RTS evidence is absent. They also verify a synthetic storage calculation:
200 MB of tablespace allocation plus 50 MB of index allocation produces 250 MB
of known allocation, with contributing evidence retained.

Some specialist health paths still handle missing locations inconsistently.
The covered examples do not establish correctness for every catalog variant.

**What I learned:** retain the source evidence and make unknown or partial
results visible alongside the calculation.

### Preserving evidence through summaries and exports

The latest visible response is not always the investigation the user wants to
export. A glossary question can follow a workflow without replacing the retained
workflow evidence. A saved regression test covers that distinction.

There are also three different views of the result: the full Python data, the
sample supplied to the model and the workbook. Tests check that observation
sampling preserves the recorded full result count. However, model-message
truncation can still remove evidence or cut structured output.

Export semantics need care. Workflow export uses retained evidence, while a
SQL-backed last-result export can rerun the query without its display row cap.
That rerun can return newer or larger results.

**What I learned:** define exactly which evidence is being explained or exported.
A model summary, a fresh query and a retained result are different artifacts.

### Improving structure while preserving behaviour

A later refactor separated implementation modules while retaining compatibility
with existing import names and launch paths. Configuration locations and
working-directory-relative output paths also needed preservation.

The approach used characterization tests: tests that capture existing behaviour
before a structural change. Saved comparisons checked selected outputs, module
identity and normalized syntax trees. Normalization accounted for the intended
import and configuration-location changes.

Existing defects were recorded separately. The structural change did not
silently fix business logic or weaken tests to make the new layout pass.

**What I learned:** paths, startup behaviour and shared state are compatibility
contracts. A refactor needs evidence about those contracts as well as readable
code organisation.

## Testing and what the results establish

The results below come from saved local validation records dated **21 October
2026**, using Python 3.12 on macOS. The records and relevant test code were
inspected for this documentation update; the application and regression suite
were **not rerun** for it.

| Recorded check | Result | Interpretation |
| --- | --- | --- |
| Final suite against the saved original source baseline | 50 passed, zero skips | The original version passed the comparison suite. |
| Same suite against the refactored application | 50 passed, zero skips | The covered contracts remained intact after the structural change. |
| Separate environment with the full dependency set | 50 passed, zero skips | The recorded dependency resolution supported the offline suite. |
| Environment without optional AI dependencies | 42 passed, 8 skipped | The covered deterministic behaviour worked; optional cases were unavailable. |
| Structural comparisons | 16 normalized syntax-tree matches and 14 matching compatibility-module identities | Supports the intended move and shared state; does not prove full behavioural equivalence. |
| Normalized output snapshots | Matched | Selected rows, warnings, traces and exported workbook contents were preserved; variable timestamps were normalized. |
| Native ODBC availability on the validation host | Failed: required native library unavailable | A real driver-dependent run was unavailable on that host. |
| Live Db2/provider integration and Windows operation | Not verified | Those environments and integrations remain outside these results. |

These are repeated runs of the comparison suite, not separate sets of 50 unique
tests. Detailed implementation and validation artifacts remain outside this
public documentation set.

The fixtures use synthetic catalog rows and provider responses, isolate
configuration and outputs, and block live network connections. Workbook checks
use the real Excel library. Covered cases include:

- Identifier rejection, parameter binding and selected row limits.
- Query failures, cursor cleanup and connection-state handling.
- Missing objects, absent evidence, calculations and tool-call ordering.
- Remembered objects and retaining workflow evidence after a later response.
- Workbook contents, warnings, query evidence and traces.
- Unsupported model tool calls and agent-call or round limits.

Tests using simulated provider replies verify selected software behaviour.
They do not measure a real model's routing accuracy or answer quality. Likewise,
mocked database results do not prove that every SQL statement is accepted by the
target Db2 environment.

Some initial test expectations were corrected after source inspection. Those
were test-authoring corrections, not product defects fixed by the refactor.
The recorded review was AI-assisted and did not constitute human operational
acceptance.

## Evaluation and measuring value

The strongest available evidence concerns selected software behaviour and
preservation through the refactor. The reviewed records do not establish
measured time savings, live-model accuracy, provider cost or end-to-end latency.

A proposed evaluation would compare three ways of completing the same approved,
non-sensitive investigations:

1. Manual investigation using existing saved queries.
2. DAISY's explicit commands and deterministic workflows.
3. DAISY's natural-language route to the same capabilities.

| Question | Evidence to collect |
| --- | --- |
| Was the right investigation selected? | Expected and actual operation, target and arguments; appropriate clarification when information is missing. |
| Are findings supported? | Each conclusion traced to returned evidence or a documented calculation; unsupported conclusions counted separately. |
| Was uncertainty handled correctly? | Missing objects, absent or stale statistics, partial failures and ambiguous requests. |
| Was the task completed more efficiently? | Time to a reviewed, usable evidence pack, including corrections and reruns. |
| Did AI improve on explicit commands? | Differences in completion rate, corrections, tool calls and evidence quality. |
| What did operation cost? | Model and prompt versions, latency, failures and token usage or cost where available. |

Cases, expected results and completion criteria should be set before timing.
Both successful and failed attempts should be included. Comparable conditions
and repeated runs would make any reported saving more meaningful.

This is proposed work. The current benefit claim is specific: DAISY brings query
selection, repeated checks and evidence assembly into one workflow. The amount
of time saved remains to be measured.

## What I learned

- **Test the hardest dependency early.** Connectivity can consume substantial
  effort before the application becomes useful.
- **Give AI a defined responsibility.** Request interpretation can be flexible
  while permitted operations, calculations and execution remain in code.
- **Treat uncertainty as part of the result.** Missing or stale evidence should
  change the conclusion.
- **Design evidence for another person to review.** Findings need supporting
  results, warnings and a trace of what ran.
- **Separate software tests from model evaluation.** A simulated tool loop and a
  real-model evaluation answer different questions.
- **Own the reasoning in AI-assisted work.** My responsibility includes
  understanding the design, questioning generated work and accepting claims
  only to the extent supported by evidence.

## Remaining limitations and priorities

The source review identifies further work on consistent missing-location
handling, alignment between AI routing paths, clarification-state expiry and
preserving complete observations for the model. Agent-call limits do not bound
every child SQL query or database execution time.

Data handling is also an operational concern. Questions, context and sampled
catalog evidence can be sent to the configured provider. Local logs and exports
can retain identifiers, query details and metadata. General redaction, retention
and export-protection controls are not implemented.

The next engineering priorities are authorised integration checks, focused
regressions for the known gaps, and the comparative evaluation above. Those are
separate from the completed packaging work and its recorded offline checks.

## Public documentation scope

This account uses generic architecture descriptions and fictional examples.
The public sharing set is the README, these engineering notes and the retained
project infographic. It excludes application source, environment configuration,
credentials, internal endpoints, organisation-specific identifiers, operational
screenshots and raw diagnostic exports.

This documents the design and available evidence; it does not claim independent
security certification or production acceptance.
