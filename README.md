# Lab 27 – YARA Rule Regression Testing

## Overview

This lab demonstrates how to perform regression testing on YARA detection rules using controlled synthetic samples.

The objective was to verify that existing detection logic continued to behave as expected when tested against positive and negative samples. Three YARA rules were created to detect synthetic downloader-related strings, Windows persistence references, and an exact custom byte sequence.

The lab also demonstrates rule compilation checks, repeatable scanning, result collection, and troubleshooting a rule-loading error.

## Lab Environment

- Operating System: Ubuntu 24.04 WSL
- YARA Version: 4.5.0
- Python Version: 3.12.3
- SHA-256 Utility: GNU coreutils 9.4

## Detection Scenarios

Three detection scenarios were tested:

1. **Downloader pattern detection:** Identifies a combination of synthetic downloader-related strings.
2. **Windows persistence detection:** Identifies references to Registry Run keys and scheduled-task commands.
3. **Custom byte-pattern detection:** Identifies an exact eight-byte sequence.

Each scenario contains one positive sample and one benign negative control.

## Testing Methodology

The following validation stages were performed:

- Verified the installed YARA and Python versions.
- Created controlled positive and negative test samples.
- Calculated SHA-256 hashes for all six samples.
- Created three independent YARA rules.
- Checked each rule for successful compilation.
- Tested every rule against its corresponding positive and negative sample.
- Executed all rules against the complete sample directory.
- Saved the final regression results.

## Results

All three rules compiled successfully.

| Detection rule | Positive sample | Negative control | Result |
|---|---|---|---|
| Downloader | Match | No match | Passed |
| Persistence | Match | No match | Passed |
| Custom byte pattern | Match | No match | Passed |

The final regression output contained three matches:

```text
Custom_Byte_Pattern_Regression samples//positive/byte-pattern.bin
Synthetic_Downloader_Regression samples//positive/downloader.txt
Synthetic_Persistence_Regression samples//positive/persistence.txt
```

The result file contained three lines, corresponding to the three expected positive detections.

## Troubleshooting

An initial attempt to scan all rules using:

```bash
yara -r rules/ samples/
```

produced:

```text
rules/(1): error: input in flex scanner failed
```

The issue occurred because YARA expects a rule file as its rule input, not a directory containing multiple rule files.

The corrected approach used a shell loop to execute each `.yar` file separately against the sample directory.

## Key Learning Outcomes

- Understood the importance of regression testing for detection rules.
- Practised validating YARA rule compilation.
- Compared positive and negative test results.
- Observed how rule changes and loading mistakes can affect testing workflows.
- Learned to collect repeatable detection results.
- Distinguished a successful pattern match from proof of malicious activity.

## Important Limitations

All samples were synthetic and created specifically for this exercise.

A successful test confirms expected behaviour against this controlled dataset. It does not establish that the rules are free from false positives or false negatives when applied to real-world files.

A YARA match indicates that the defined pattern was identified. It does not independently prove malware execution, persistence, or compromise.
