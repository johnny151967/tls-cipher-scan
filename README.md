# tls-cipher-scan
.github/workflows/tls-scan.yml
name: TLS Cipher Scan

on:
  workflow_dispatch:
    inputs:
      target:
        description: "Enter target hostname and port (e.g. example.com:443)"
        required: true
        default: "example.com:443"

jobs:
  scan:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          pip install --upgrade pip
          pip install sslyze

      - name: Run SSLyze scan
        run: |
          mkdir results
          python -m sslyze --json_out=results/scan.json ${{ github.event.inputs.target }}

      - name: Parse for weak ciphers
        run: |
          python scripts/parse_weak_ciphers.py results/scan.json

      - name: Upload report artifact
        uses: actions/upload-artifact@v4
        with:
          name: TLS-Scan-Report
          path: results/scan.json
