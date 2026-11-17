# Timeline – Lab 27: YARA Rule Regression Testing

## Activity Timeline

### 1. Environment Preparation

- Created the Lab 27 workspace.
- Created separate directories for rules, positive samples, negative samples, results, and tools.
- Verified YARA 4.5.0, Python 3.12.3, and GNU coreutils 9.4.

### 2. Synthetic Sample Creation

- Created a downloader-related positive sample.
- Created a Windows persistence-related positive sample.
- Generated an exact eight-byte positive pattern sample.
- Created two benign text controls.
- Generated a byte-pattern control with one modified byte.

### 3. Sample Integrity Verification

- Listed all six samples and verified their sizes.
- Calculated SHA-256 hashes.
- Recorded the hashes for repeatable sample identification.

### 4. YARA Rule Development

- Created `downloader.yar`.
- Created `persistence.yar`.
- Created `byte_pattern.yar`.
- Checked all three rules against `/dev/null`.
- Confirmed successful compilation with exit status `0` for each rule.

### 5. Individual Regression Tests

- Downloader rule matched its positive sample.
- Downloader rule did not match its benign control.
- Persistence rule matched its positive sample.
- Persistence rule did not match its benign control.
- Byte-pattern rule matched its exact positive sequence.
- Byte-pattern rule did not match the modified control sequence.

### 6. Troubleshooting

- Attempted to execute all rules using `yara -r rules/ samples/`.
- Observed an input scanner error because a directory was supplied instead of an individual rule file.
- Replaced the command with a shell loop over `rules/*.yar`.
- Successfully executed each rule against the sample directory.

### 7. Final Result Collection

- Redirected the corrected regression output to `results/regression-results.txt`.
- Verified that the output contained three lines.
- Confirmed that all three expected positive samples matched.

## Final Status

**Regression testing completed successfully against the controlled synthetic dataset.**

- Rules compiled: 3
- Positive samples: 3
- Negative samples: 3
- Expected matches: 3
- Unexpected negative-control matches: 0
- Final result lines: 3

The findings are limited to the synthetic samples used in this lab.
