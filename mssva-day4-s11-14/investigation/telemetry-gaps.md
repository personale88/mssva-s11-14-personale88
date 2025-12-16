# Telemetry Gaps

The system fails to emit logs or warnings when record parsing fails.
Parsing loops break silently on fread or allocation failures.

This creates blind spots where operators cannot distinguish malformed
inputs from successful processing.
