# Troubleshooting Notes – Lab 27: YARA Rule Regression Testing

## Issue 1 – YARA Rule Directory Loading Error

### Command executed

```bash
yara -r rules/ samples/
```

### Error observed

```text
rules/(1): error: input in flex scanner failed
```

The same error occurred when attempting to redirect the output:

```bash
yara -r rules/ samples/ > results/regression-results.txt
```

The output file was empty:

```text
0 results/regression-results.txt
```

### Cause

YARA expects its first positional argument to identify a rule file or compiled rule input.

The `-r` option enables recursive scanning of the target directory. It does not recursively load multiple rule files from a directory.

Therefore, `rules/` could not be used directly as the rule input.

### Resolution

A shell loop was used to execute each YARA rule file separately:

```bash
for rule in rules/*.yar; do
    echo "===== Testing $rule ====="
    yara -r "$rule" samples/
done
```

### Verification

The corrected command produced three expected detections:

```text
Custom_Byte_Pattern_Regression samples//positive/byte-pattern.bin
Synthetic_Downloader_Regression samples//positive/downloader.txt
Synthetic_Persistence_Regression samples//positive/persistence.txt
```

The results were saved using:

```bash
for rule in rules/*.yar; do
    yara -r "$rule" samples/
done > results/regression-results.txt
```

The final line count was:

```text
3 results/regression-results.txt
```

The issue was resolved without modifying the rules or reinstalling YARA.

---

## Issue 2 – Empty Output from Negative Tests

### Observation

The negative test commands returned no output.

For example:

```bash
yara rules/downloader.yar samples/negative/benign-downloader.txt
```

### Explanation

YARA normally prints a result when a rule matches a target file.

When no rule matches, it produces no standard output. This is expected behaviour and does not automatically indicate a scanning failure.

### Verification

The corresponding positive test produced the expected rule name and sample path.

The negative test produced no match, which was the intended outcome.

---

## Issue 3 – Understanding the Final Result Count

### Command

```bash
wc -l results/regression-results.txt
```

### Observed result

```text
3 results/regression-results.txt
```

### Explanation

The three output lines corresponded to the three positive samples.

The negative controls did not produce matches.

This confirms the expected behaviour for the six samples included in this exercise.

It does not establish that the rules will never produce false positives or false negatives against other files.

---

## Troubleshooting Summary

| Issue | Cause | Resolution |
|---|---|---|
| Rule directory loading error | Directory supplied where YARA expected a rule file | Execute each `.yar` file using a shell loop |
| Empty negative-test output | No rule matched | Confirmed as expected behaviour |
| Empty result file after failed scan | Rule-loading command failed | Re-ran using individual rule files |
| Three final matches | Each positive sample triggered its corresponding rule | Confirmed expected regression outcome |

## Key Learning

Understanding command-line argument behaviour is important when automating security testing.

A successful regression workflow must validate not only detection logic but also rule loading, scanning execution, output collection, and interpretation.
