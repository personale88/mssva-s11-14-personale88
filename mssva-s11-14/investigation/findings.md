Title: Untrusted record_count breaks processing assumptions
Flag: Design Assumption Broken
Location (file:function): parser.c:parse_header
Instrumentation Used: sec_log, sec_info
Trigger Condition: Input file declares a large record_count that does not match actual records.
Root Cause: The system trusts record_count from the input header without validating it against file size or actual parsed records.
Impact: The application processes zero records while reporting a large record_count, leading to silent data loss and incorrect system state.
Evidence: Runtime output shows [SEC]record_count:29541 with records_processed=0.
Title: Silent record parsing failures lack telemetry
Flag: Telemetry Gap Identified
Location (file:function): record.c:parse_records
Instrumentation Used: sec_log, sec_info
Trigger Condition: fread or memory allocation failure while parsing records.
Root Cause: The parsing loop breaks on errors without emitting any warning or error telemetry.
Impact: Record parsing failures occur without operator visibility, making data loss and corruption difficult to detect.
Evidence: Runtime output shows header telemetry but no logs explaining why records_processed remains zero.
Title: Off-by-one read violates payload memory ownership
Flag: Memory Ownership Violation
Location (file:function): memory.c:process_record
Instrumentation Used: AddressSanitizer, assertion on record length
Trigger Condition: FAST_MODE enabled with record payload of length N.
Root Cause: The code copies rec->length + 1 bytes from a payload buffer that was only allocated with rec->length bytes.
Impact: Out-of-bounds memory reads can leak adjacent memory contents or cause crashes, leading to undefined behavior and potential security exposure.
Evidence: Code inspection shows memcpy(buf, rec->payload, rec->length + 1); which exceeds the allocated payload size and is detectable with ASan under fuzzed inputs.
Title: Record processing failure occurs without error signaling
Flag: Silent Failure Detected
Location (file:function): stats.c:stats_dump / main.c:main
Instrumentation Used: stats_dump output, runtime telemetry
Trigger Condition: Input file results in zero valid records being processed.
Root Cause: The application does not treat zero processed records as an error condition and emits no warning or failure signal.
Impact: The system silently produces incomplete results while appearing to execute successfully, risking unnoticed data loss in production workflows.
Evidence: Runtime output shows records_processed=0 with normal program termination and no warning or error logs.
Title: FAST_MODE off-by-one memory read exposed by fuzzed input
Flag: Fuzzer-Only Bug Explained
Location (file:function): memory.c:process_record
Instrumentation Used: AddressSanitizer, record length assertion
Trigger Condition: Fuzzed input producing oversized record lengths with FAST_MODE enabled.
Root Cause: The code copies rec->length + 1 bytes from a payload buffer allocated with only rec->length bytes, which becomes observable with abnormal input sizes.
Impact: Out-of-bounds memory reads can cause crashes or leak adjacent memory, representing a serious security risk that is unlikely to surface under normal inputs.
Evidence: Oversized fuzzed inputs trigger assertion failures or ASan reports pointing to memcpy in process_record.
