# trivy scan 





name: Trivy Security Scan for NGINX

&#x20;

on:

&#x20; push:

&#x20;   branches:

&#x20;     - main

&#x20; pull\_request:

&#x20;   branches:

&#x20;     - main



jobs:

&#x20; build:

&#x20;   name: Scan nginx:latest with Trivy

&#x20;   runs-on: ubuntu-20.04



&#x20;   steps:

&#x20;     - name: Checkout repository

&#x20;       uses: actions/checkout@v3



&#x20;     - name: Set Date

&#x20;       id: set-date

&#x20;       run: echo "DATE=$(date +%m.%d.%Y)" >> $GITHUB\_ENV



&#x20;     - name: Create output directory

&#x20;       run: mkdir -p "GitHub Actions/Trivy Automation"



&#x20;     - name: Install Trivy

&#x20;       run: |

&#x20;         sudo apt-get update

&#x20;         sudo apt-get install -y curl

&#x20;         curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sudo sh -s -- -b /usr/local/bin v0.57.0



&#x20;     - name: Log in to GitHub Container Registry

&#x20;       run: echo "${{ secrets.GHCR\_PAT }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin



&#x20;     - name: Download Trivy vulnerability database

&#x20;       run: trivy image --download-db-only



&#x20;     - name: Run Trivy vulnerability scanner

&#x20;       run: |

&#x20;         OUTPUT\_FILE="GitHub Actions/Trivy Automation/scan-results-${DATE}.json"

&#x20;         mkdir -p "GitHub Actions/Trivy Automation"

&#x20;         trivy image --format json --ignore-unfixed --vuln-type os,library --severity CRITICAL,HIGH,MEDIUM,LOW -o "$OUTPUT\_FILE" nginx:latest



&#x20;     - name: Verify scan results exist

&#x20;       run: ls -l "GitHub Actions/Trivy Automation/"



&#x20;     - name: Commit scan results to repository

&#x20;       env:

&#x20;         GITHUB\_TOKEN: ${{ secrets.GITHUB\_TOKEN }}

&#x20;       run: |

&#x20;         git config --global user.name "GitHub Actions"

&#x20;         git config --global user.email "actions@github.com"

&#x20;         git add "GitHub Actions/Trivy Automation/scan-results-${DATE}.json"

&#x20;         git commit -m "Add Trivy scan results (${DATE})"

&#x20;         git push origin HEAD:main

