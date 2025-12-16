# Fuzzing Notes

Fuzzing revealed an off-by-one memory read in FAST_MODE when record
payload lengths exceeded expected bounds.

This bug was not observable under normal inputs and became visible
only when fuzzed inputs triggered abnormal record sizes.
