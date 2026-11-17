# Investigation Notes
## 1. Investigation Objective

The objective was to validate three YARA detection rules using controlled positive and negative samples.

The tests focused on verifying expected detection behaviour, confirming that benign controls did not trigger the rules, and recording the final results.

## 2. Environment Verification

The following tool versions were confirmed:

| Component | Version |
|---|---|
| Operating System | Ubuntu 24.04 WSL |
| YARA | 4.5.0 |
| Python | 3.12.3 |
| GNU coreutils | 9.4 |

All required tools were available.

## 3. Sample Preparation

Six synthetic samples were created.

### Positive samples

| Sample | Detection objective |
|---|---|
| `downloader.txt` | Contains downloader-related strings |
| `persistence.txt` | Contains Windows persistence references |
| `byte-pattern.bin` | Contains the exact eight-byte test sequence |

### Negative samples

| Sample | Purpose |
|---|---|
| `benign-downloader.txt` | Benign text without the required downloader indicators |
| `benign-persistence.txt` | Benign text without the required persistence references |
| `byte-pattern-control.bin` | Contains a modified byte sequence |

The positive byte sequence was:

```text
C0 DE 24 A1 7F 55 0D F0
```

The negative control sequence was:

```text
C0 DE 24 A1 33 55 0D F0
```

The fifth byte was changed from `7F` to `33`, preventing an exact match against the rule.

## 4. Sample Integrity

SHA-256 hashes were calculated for all six samples.

| Sample | SHA-256 |
|---|---|
| `benign-downloader.txt` | `b6ee30b0b0ede05fb99c8f6eb8d52cef547a8ab98c3973c2840f2a1592a9e250` |
| `benign-persistence.txt` | `f3dfdf04860eb68f253fefedaa62bbeac2908df3a21ac1e8646604077faf8342` |
| `byte-pattern-control.bin` | `6a2f1b2fcecee12ca418271b4ef64ea128313a0e43e7de7a7d72180963415a13` |
| `byte-pattern.bin` | `f2a26fac1bd0f34ec386dc711d327c9bdf5fdbb8f7cd00750a86f5384f03ae6a` |
| `downloader.txt` | `f1209f811977c706c28c670748aff9141915a7537029c1a7ba802d539a8a01b3` |
| `persistence.txt` | `5e00a5166460fee4c3fcfc7c1223a923c022dad57e01db3f95c2` |

## 5. Rule Validation

Three independent YARA rule files were created:

- `downloader.yar`
- `persistence.yar`
- `byte_pattern.yar`

Each rule was checked using:

```bash
for rule in rules/*.yar; do
    echo "Checking: $rule"
    yara "$rule" /dev/null
    echo "Exit status: $?"
done
```

All three rules returned exit status `0`.

No compilation errors were observed.

## 6. Individual Test Results

### Downloader rule

Positive test:

```bash
yara rules/downloader.yar samples/positive/downloader.txt
```

Observed:

```text
Synthetic_Downloader_Regression samples/positive/downloader.txt
```

Negative test:

```bash
yara rules/downloader.yar samples/negative/benign-downloader.txt
```

Observed: No output.

### Persistence rule

Positive test:

```bash
yara rules/persistence.yar samples/positive/persistence.txt
```

Observed:

```text
Synthetic_Persistence_Regression samples/positive/persistence.txt
```

Negative test:

```bash
yara rules/persistence.yar samples/negative/benign-persistence.txt
```

Observed: No output.

### Custom byte-pattern rule

Positive test:

```bash
yara rules/byte_pattern.yar samples/positive/byte-pattern.bin
```

Observed:

```text
Custom_Byte_Pattern_Regression samples/positive/byte-pattern.bin
```

Negative test:

```bash
yara rules/byte_pattern.yar samples/negative/byte-pattern-control.bin
```

Observed: No output.

## 7. Final Regression Execution

The three rule files were executed individually against the complete sample directory:

```bash
for rule in rules/*.yar; do
    yara -r "$rule" samples/
done > results/regression-results.txt
```

The saved output was:

```text
Custom_Byte_Pattern_Regression samples//positive/byte-pattern.bin
Synthetic_Downloader_Regression samples//positive/downloader.txt
Synthetic_Persistence_Regression samples//positive/persistence.txt
```

The output was verified using:

```bash
wc -l results/regression-results.txt
```

Observed:

```text
3 results/regression-results.txt
```

## 8. Final Assessment

| Validation | Result |
|---|---|
| Rule compilation | Passed |
| Downloader positive test | Match |
| Downloader negative test | No match |
| Persistence positive test | Match |
| Persistence negative test | No match |
| Byte-pattern positive test | Match |
| Byte-pattern negative test | No match |
| Final regression output | 3 matches |

All expected test outcomes were observed.

