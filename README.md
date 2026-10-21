# DAISY — Db2 AI Support for You

**A Db2 for z/OS investigation assistant that brings catalog evidence, repeatable
checks and reviewable findings into one terminal workflow.**

DAISY, also called the **DB2AI Catalog Console**, helps database administrators
understand database objects, investigate dependencies and storage, and prepare
evidence for review. It brings together catalog queries that would otherwise need
to be run and interpreted separately.

This is public documentation of an internal enterprise project. Examples are
fictional. Source code, internal configuration and operational data are outside
the public sharing scope.

## Why DAISY exists

A routine DBA question can lead to several separate tasks: identify the object,
find the relevant catalog information, follow its dependencies, interpret
statistics and assemble the findings. Repeating that process takes time and makes
the result depend heavily on an individual's knowledge and saved queries.

DAISY provides a consistent starting point. The user enters a supported command
or question; Python retrieves metadata, runs the selected investigation and
presents evidence. Optional AI helps interpret questions and select available
tools. The database queries and investigation calculations remain in Python.

## Architecture at a glance

![DAISY overview showing DBA questions, Python routing, validation, predefined catalog tools, Db2 evidence, investigation workflows, terminal results and Excel exports.](assets/DB2AI_Catalog_Console_DAISY_Infographic.png)

[Open the full-size infographic](assets/DB2AI_Catalog_Console_DAISY_Infographic.png).

The infographic groups the main responsibilities; its numbers are conceptual
labels. In execution, workflows call tools that query Db2, and checks run at
multiple input and dispatch points. AI routing is optional.

1. **Ask or select a command.** A DBA supplies a question or an explicit operation,
   with an object or investigation scope.
2. **Resolve the request.** Python handles commands and recognised local routes.
   When configured, a large language model (LLM) can interpret intent and propose
   a supported tool.
   Missing details may lead to clarification.
3. **Validate and dispatch.** The application checks the selected operation and
   its arguments through the relevant dispatch path. Predefined tools contain
   the SQL; workflows prescribe the investigation steps.
4. **Collect and interpret evidence.** Catalog and real-time statistics (RTS)
   reads return metadata. Python combines results, calculations and warnings.
   Local glossary and pasted-output interpretation can work without a database
   query.
5. **Review and retain results.** The terminal shows tables and findings. Session
   context supports follow-up questions, and Excel export can retain supporting
   evidence for review.

## What DAISY can do

| Investigation need | Implemented capability |
| --- | --- |
| Understand an object | Inspect definitions, columns, indexes, constraints and privileges. |
| Assess change impact | Trace catalog-recorded packages, plans, views, triggers and related objects. |
| Investigate storage | Combine tablespace, index and RTS evidence into object, database and storage-group summaries. |
| Review health and readiness | Assemble prescribed checks for status, partitions, pending definitions and supported object families. Findings remain advisory. |
| Explain and compare | Explain metadata terms and status codes, interpret pasted DISPLAY output, and compare supported columns, indexes and catalog-derived DDL. |
| Prepare a handover | Present findings with warnings and tool traces, and export results to Excel. |

These capabilities are supported by the inspected implementation and selected
offline tests. Live compatibility and coverage vary by catalog, object type,
configuration and entry path.

## Practical benefits and time savings

| Manual effort | How DAISY reduces repeated work |
| --- | --- |
| Recall and assemble several catalog queries | Predefined tools provide reusable investigation operations. |
| Repeat the same sequence of checks | Workflows run prescribed steps and combine their results. |
| Reconcile findings across separate outputs | Structured results keep contributing evidence, warnings and calculations together. |
| Copy results into a handover document | Excel evidence export packages investigation material for review. |

The intended time saving comes from reducing query preparation, repeated
navigation and evidence assembly. Saved queries and ordinary scripts remain a
useful comparison baseline; AI's additional value is easier access through
natural language.

**The reviewed records do not establish measured minutes saved or a productivity
percentage.** A fair comparison would time the same investigations manually,
through explicit DAISY commands and through its AI route, while checking result
quality. The [engineering notes](engineering-notes.md#evaluation-and-measuring-value)
describe that proposed evaluation.

## A small example

After authorised setup and login, a DBA could investigate the fictional object
`DEMO.T_ACCOUNT`:

```text
/metadata DEMO.T_ACCOUNT
/export-evidence
/quit
```

The metadata workflow gathers an object profile and supporting results.
The export command saves the retained workflow evidence to Excel. The DBA
reviews the findings and any missing evidence before deciding what to do next.
Natural-language questions provide another entry point when AI is configured;
their routing quality still needs live evaluation.

## Scope, controls and data handling

The exposed operations investigate catalog metadata and interpret supplied text.
They do not provide a general SQL execution tool, execute database changes or
utilities, retrieve application business rows, or issue live DISPLAY commands.
Displaying or comparing Data Definition Language (DDL) text does not execute it.

Code checks include approved tool dispatch, argument and identifier validation,
SQL keyword checks and limits on the native agent's rounds and tool calls.
These controls have specific limits: keyword checks are not a full SQL parser,
and the connection layer does not itself establish database-enforced read-only
access. Account permissions and authorised configuration remain essential.

Catalog evidence can be incomplete or stale. Dependency results do not establish
every runtime dependency, and readiness findings do not replace operational
approval. With AI enabled, questions, context and sampled evidence can reach the
configured provider. Logs and exports can retain sensitive metadata; automatic
redaction and retention controls are not implemented.

## Environment and validation

An authorised installation needs Python, a compatible native Db2 ODBC driver,
a configured data source and catalog access. Excel export needs its Python
dependency; AI adds optional model-client dependencies and an approved provider.
The public documentation set does not include a runnable installation.

Saved validation records from **21 October 2026** report **50 passing offline
tests** against the refactored application, using synthetic data and mocked
database/provider boundaries on Python 3.12 and macOS. Live Db2/provider
integration and Windows operation were not verified in that validation.

Read [Engineering Notes](engineering-notes.md) for the design decisions,
development challenges, contribution and AI-assistance account, test evidence,
lessons learned and remaining work.
