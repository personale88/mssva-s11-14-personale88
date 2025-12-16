## System Overview

dataproc-agent is an internal batch preprocessing utility designed to read a
proprietary binary input file and prepare records for ingestion.

## Input Model

The program expects a binary file containing a header followed by multiple
records. The header specifies the record count and flags, which are trusted
by the system.

## Processing Flow

1. Load configuration from environment variables (e.g., DATAPROC_FAST)
2. Open the input file
3. Parse header fields including version, record_count, and flags
4. Parse records based on the declared record_count
5. Validate and process each record
6. Emit telemetry and statistics

## Assumptions

- The input file is well-formed and consistent
- Record counts and lengths are trusted
- Environment variables are safe and controlled
- Memory allocations succeed

## Security-Relevant Observations

Input validation is minimal and several design assumptions are implicit rather
than enforced. Internal state is exposed via telemetry output.
