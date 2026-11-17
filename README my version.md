# YARA-Rule-Regression-Testing
## Overview
### 1. What Is Regression Testing?

Regression testing is the process of rechecking existing functionality after changes are made to ensure that previously working behaviour has not been broken.

In cybersecurity detection engineering, regression testing is used to verify that detection rules continue to identify intended patterns and do not unexpectedly trigger on benign test data.

For YARA, this means repeatedly scanning known test samples and comparing actual matches with expected results.

### 2. Why Is It Important?

A YARA rule may initially detect a particular pattern correctly. However, changes to its strings, conditions, or matching logic can introduce problems.

Examples include:

- A rule stops detecting a previously identified pattern.
- A rule begins matching unrelated benign files.
- A required string is accidentally removed.
- A condition becomes too restrictive.
- A wildcard makes a byte-pattern rule overly broad.
- A rule fails to compile after an edit.

Regression testing helps identify these problems before rules are deployed into a larger detection workflow.

### 3. Positive and Negative Testing

#### Positive testing

A positive sample contains the pattern that a rule is designed to detect.

Expected result: YARA reports a match.

#### Negative testing

A negative sample does not contain the required detection pattern.

Expected result: YARA produces no match.

Both test types are important. Testing only positive samples can demonstrate that a rule detects one known pattern, but it does not establish whether the rule also matches unrelated files.

### 4. False Positives and False Negatives

**False positive:** A benign file matches a detection rule unexpectedly.

**False negative:** A sample that should match a detection rule does not produce a match.

Regression testing helps identify these problems within the available test dataset.

A small synthetic dataset cannot establish the real-world false-positive or false-negative rate of a rule.

### 5. YARA Rule Compilation

Before executing a rule, its syntax should be validated.

Example:

```bash
yara rules/downloader.yar /dev/null
```

A successful execution with exit status `0` confirms that the rule was accepted.

Because `/dev/null` contains no test data, no detection output is expected.

Compilation validation and detection validation are separate activities.

### 6. Recursive Scanning

YARA supports recursive scanning of files inside a target directory.

Example:

```bash
yara -r rules/downloader.yar samples/
```

Here:

- `rules/downloader.yar` is the rule file.
- `samples/` is the target directory.
- `-r` enables recursive scanning.

When multiple rule files are maintained separately, a shell loop can execute each one:

```bash
for rule in rules/*.yar; do
    yara -r "$rule" samples/
done
```

### 7. Regression Result Collection

Test results can be redirected to a file:

```bash
for rule in rules/*.yar; do
    yara -r "$rule" samples/
done > results/regression-results.txt
```

The result file can then be reviewed:

```bash
cat results/regression-results.txt
```

And its number of matching lines can be checked:

```bash
wc -l results/regression-results.txt
```

### 8. Sample Integrity with SHA-256

SHA-256 hashes help identify the exact samples used during testing.

Example:

```bash
find samples -type f -print0 | sort -z | xargs -0 sha256sum
```

If a sample changes, its hash will generally change as well. This supports repeatability and evidence tracking.

### 9. Lab 27 Application

This lab applied regression testing to three YARA detection scenarios:

1. Synthetic downloader-related strings.
2. Synthetic Windows persistence references.
3. An exact custom byte sequence.

Each scenario included one positive sample and one negative control.

All three rules compiled successfully and produced the expected results against the six controlled samples.

### 10. Important Limitation

A YARA match establishes that a rule's defined pattern was found in a file.

It does not independently establish that the file is malicious, that a command was executed, or that persistence or compromise occurred.

Regression testing validates rule behaviour against the test dataset, not the complete effectiveness of a detection rule in every environment.

This lab demonstrates how to perform regression testing on YARA detection rules using controlled synthetic samples.

The objective was to verify that existing detection logic continued to behave as expected when tested against positive and negative samples. Three YARA rules were created to detect synthetic downloader-related strings, Windows persistence references, and an exact custom byte sequence.

The lab also demonstrates rule compilation checks, repeatable scanning, result collection, and troubleshooting a rule-loading error.

## Lab Environment

- Operating System: Ubuntu 24.04 WSL
- YARA Version: 4.5.0
- Python Version: 3.12.3
- SHA-256 Utility: GNU coreutils 9.4

## Lab Objectives

### Primary Objective

To develop a repeatable testing process for validating YARA detection rules and ensuring that their expected matching behaviour remains consistent across controlled test cases.

### Specific Objectives

1. **Validate rule syntax:** Confirm that each YARA rule compiles successfully before testing.

2. **Build a test dataset:** Prepare synthetic positive samples and benign negative controls for multiple detection scenarios.

3. **Test string-based detection:** Verify that downloader-related indicators are identified when all required strings are present.

4. **Test persistence-related detection:** Validate detection of Registry Run key and scheduled-task references.

5. **Test binary pattern matching:** Confirm that an exact byte sequence is detected while a modified sequence is not.

6. **Verify negative behaviour:** Ensure that the benign controls do not unexpectedly trigger the corresponding rules.

7. **Automate repeated scans:** Use a shell loop to execute multiple rule files against a common sample directory.

8. **Preserve test evidence:** Record sample hashes and save the final regression output.

9. **Troubleshoot execution issues:** Identify and resolve incorrect rule-loading syntax without unnecessarily changing valid detection logic.

10. **Interpret findings accurately:** Distinguish successful pattern detection from proof of malicious behaviour.

### Expected Learning Outcomes

After completing this lab, the learner should be able to:

- Explain why detection rules require regression testing.
- Differentiate positive and negative test cases.
- Validate YARA rule compilation.
- Execute multiple YARA rules systematically.
- Identify unexpected detection behaviour within a controlled dataset.
- Preserve repeatable test evidence using SHA-256 and output files.
- Explain the limitations of synthetic regression testing.

## Lab Scenario 

### Scenario

A security analyst maintains a collection of YARA rules used to identify suspicious file patterns during malware triage.

Before making these rules available for broader scanning, the analyst wants to confirm that the existing detection logic behaves as expected.

Three detection scenarios have been selected for validation:

- A downloader-related pattern containing a URL and several file and process operation strings.
- A Windows persistence-related pattern containing a Registry Run key reference and scheduled-task commands.
- A custom binary pattern consisting of a specific eight-byte sequence.

The analyst prepares a controlled test collection containing one positive sample and one benign negative control for each scenario.

The positive samples are expected to trigger their corresponding rules. The negative controls should not produce matches.

### Investigation Approach

The analyst first verifies the installed YARA version and prepares separate directories for rules, samples, and results.

Each rule is created independently and checked for successful compilation.

Individual positive and negative tests are performed to establish the expected detection behaviour.

The analyst then attempts to scan the entire sample collection using a single command. The command fails because a directory containing multiple rule files was supplied where YARA expected a rule file.

Rather than modifying the detection logic, the analyst reviews the command structure and changes the execution method to process each `.yar` file separately.

The corrected workflow runs all three rules against the complete sample directory and stores the matching results in a text file.

### Investigation Questions

1. Do all three YARA rules compile successfully?
2. Does each rule detect its intended positive sample?
3. Do the benign controls remain unmatched?
4. Does the custom byte-pattern rule distinguish the original sequence from the modified control?
5. Can multiple rule files be executed systematically against one sample directory?
6. Are the final results consistent with the expected test outcomes?

### Expected Outcome

The regression exercise should produce three expected detections, one for each positive sample, without matching any of the three negative controls.

The analyst should also retain sample hashes, rule files, and regression output as evidence of the testing process.

### Scope and Limitations

This scenario uses synthetic files and controlled indicators. No actual downloader, scheduled task, Registry modification, or malicious executable is involved.

The exercise validates the behaviour of the selected rules against a small test dataset. It does not represent a complete malware-analysis investigation or establish the effectiveness of the rules against all real-world threats.

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

