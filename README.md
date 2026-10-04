# Password Cracker

> Password-hash cracking utility for **authorized security research, lab work, and owned data only**.

## Overview

The project is structured as a reusable Python package with separate layers for core attack logic, orchestration, interfaces, and utilities.

Current attack strategies include dictionary, rule-based, mask, and bounded brute-force attacks. Jobs can run through the CLI or the local HTTP API, and the watcher can process incoming hash files.

## Capabilities

### Attack strategies

- Dictionary attack
- Rule-based mangling
- Mask attack using patterns such as `?l?l?d?d`
- Brute-force search up to a configured maximum length
- Configurable attack order

### Supported verification algorithms

- MD5
- SHA-1
- SHA-256
- SHA-512
- SHA3-256
- SHA3-512
- bcrypt
- Argon2

### Operational features

- Multiprocessing support across attack engines
- Built-in common-password candidates
- Wordlist discovery under `wordlist/`
- Resume configuration
- Job-specific cracked/failed result files
- Interactive and non-interactive CLI modes
- Folder watch mode
- Local-only Flask API by default

## Important hash-detection detail

SHA-256 and SHA3-256 both produce 64-character hexadecimal digests. SHA-512 and SHA3-512 both produce 128-character hexadecimal digests.

Because digest length alone cannot distinguish those pairs, the tool intentionally auto-detects the SHA-2 variant for those ambiguous lengths. Use an explicit algorithm option when working with a SHA-3 hash.

## Installation

From the repository root:

```bash
python install.py --upgrade-pip
```

or:

```bash
python -m pip install -r requirements.txt
```

## CLI

Interactive mode:

```bash
python -m cracker.cli
```

Non-interactive example:

```bash
python -m cracker.cli \
  --hash 5d41402abc4b2a76b9719d911017c592 \
  --wordlist wordlist/rockyou.txt \
  --maxlen 5
```

Customize attack order:

```bash
python -m cracker.cli \
  --hash 5d41402abc4b2a76b9719d911017c592 \
  --mask "?l?l?d?d" \
  --attack-order dictionary,rules,mask,bruteforce
```

## HTTP API

The Flask API is restricted to local requests by default.

Start it from the repository root with the command documented by the current Flask application:

```bash
flask --app cracker.api run
```

Example request:

```bash
curl -X POST http://127.0.0.1:5000/crack \
  -H "Content-Type: application/json" \
  -d '{
    "hash": "5d41402abc4b2a76b9719d911017c592",
    "wordlist": "wordlist/rockyou.txt",
    "maxlen": 5,
    "use_multiprocessing": false,
    "enable_mask": true,
    "mask_patterns": ["?l?l?d?d"]
  }'
```

## Safety and security

Use this tool only on hashes and files you own or are explicitly authorized to test.

Current defensive controls include:

- Local-only API behavior by default
- Wordlist path restrictions unless external-wordlist access is explicitly enabled
- Hash-file path restrictions unless external access is explicitly enabled
- Debug mode disabled by default when the API is started directly

Cracking can be extremely CPU-intensive, especially for brute-force searches. Use conservative limits and test in controlled environments.

## Testing

Run:

```bash
pytest
```

The test suite covers core hashing/attack behavior plus application, CLI, API, and tool wiring.

## Architecture

```text
cracker/
├── core.py      # hashing + attack engines
├── app.py       # job orchestration + persistence
├── api.py       # Flask API
├── cli.py       # command-line interface
└── tools.py     # utility/watch functionality
```

The repository also contains the top-level launcher, installation helpers, tests, and wordlists.

## Limitations

- Brute-force search grows exponentially with maximum length and charset size.
- Multiprocessing uses the available CPU resources and can create significant load.
- Hash identification from a digest string is best-effort; explicit algorithm selection is preferred when the format is ambiguous.
- This is a security-learning tool, not a commercial password-recovery service.

## License

See [LICENSE](LICENSE).
