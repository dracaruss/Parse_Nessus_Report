# nessus2report

nessus2report turns one or two Nessus scan exports into a finished Abacode EIVA report in Word format, plus the customer folder of HTML reports that Parse-Nessus.ps1 used to produce. It replaces the old workflow of exporting five files, running Auto-Rename.ps1 and Parse-Nessus.ps1, and copying findings out of the HTML by hand.

There are two versions that behave identically and use the same flags. Use `nessus2report.sh` on Kali and `nessus2report.ps1` on Windows. The report template is built into each script, so the script is the only file you need, and neither version installs anything on your machine.

## What you need from Nessus

Export each scan in the .nessus format and nothing else. The .nessus file already contains everything the report needs.

## Setup on Kali

Download the script and make it executable.

```bash
chmod +x nessus2report.sh
```

The script needs python3-lxml, which comes with Kali.

## Setup on Windows

Windows blocks scripts downloaded from the internet, so the first time you run the script you will likely see an error saying it "is not digitally signed". Open PowerShell in the folder that contains the script and run these two commands once.

```powershell
Unblock-File .\nessus2report.ps1
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

The first command removes the "downloaded from the internet" mark from the file. The second command lets you run local scripts under your own user account, and it does not need administrator rights. After that, the script runs normally as `.\nessus2report.ps1`.

Every time you download a new copy of the script, run `Unblock-File .\nessus2report.ps1` again, because the fresh download is marked as blocked too. The execution policy command only needs to be run once.

If your machine is managed by a company Group Policy that forces script signing, the second command will be refused. In that case, run the script through a one-time bypass instead.

```powershell
powershell -ExecutionPolicy Bypass -File .\nessus2report.ps1 -i .\internal.nessus -c "Acme Health Inc" -a ACME -r assessment
```

If Microsoft Word is installed, the Windows version has Word set the exact Table of Contents page numbers.

## Example 1: Internal scan only

```bash
./nessus2report.sh -i internal.nessus -c "Acme Health Inc" -a ACME -r assessment
```

```powershell
.\nessus2report.ps1 -i .\internal.nessus -c "Acme Health Inc" -a ACME -r assessment
```

This fills the Internal (IVA) sections and removes the External sections from the report.

## Example 2: Internal and External scans together

```bash
./nessus2report.sh -e external.nessus -i internal.nessus -c "Acme Health Inc" -a ACME -r assessment
```

```powershell
.\nessus2report.ps1 -e .\external.nessus -i .\internal.nessus -c "Acme Health Inc" -a ACME -r assessment
```

This fills the External (EVA) and Internal (IVA) sections in a single report, with every finding from both scans listed in the Table of Contents.

## Example 3: Just run it and answer the prompts

```bash
./nessus2report.sh
```

```powershell
.\nessus2report.ps1
```

The script asks for the external file and the internal file, and you press Enter to skip either one. It then asks for the customer name, the abbreviation, whether this is an assessment or a scan, and whether to skip Informational findings.

If a file name contains spaces, wrap it in quotes, for example `-i '.\VULN Scan Sept 2026.nessus'`.

## Options

| Flag | Meaning |
|------|---------|
| `-e` | External .nessus file |
| `-i` | Internal .nessus file |
| `-c` | Customer full name |
| `-a` | Customer abbreviation, used in the header and body text |
| `-r` | `assessment` or `scan`. An assessment produces the full report with a detailed page for every finding. A scan produces the shorter Executive Summary report with quarter dates and no detailed findings pages. |
| `-d` | Custom cover date. The default is today's date. |
| `-o` | Output file path. The default is a file next to the .nessus file, named like `ACME EIVA 2026 Q3.docx` for an assessment or `Abacode_Acme_Health_Inc_Executive_Summary_Report_Q3_2026.docx` for a scan. |
| `-n` | Skip Informational findings without asking |
| `-h` | Show the built-in help |

Any option you leave out is asked for when the script runs.

## What the script does for you

The script fills in the customer name and abbreviation everywhere, including the header, and puts today's date on the cover. It counts findings by severity, builds the findings tables, and writes a detailed page for each finding, with every finding starting on a new page. Each detailed page includes the description, the recommendation, and the affected assets listed as bullets.

All SSL and TLS findings are combined into one "Multiple SSL/TLS Issues" finding. SMBv1 is raised to High, which matches the old Parse-Nessus behavior. A section with no findings shows "No Notable Findings". The scope table in Appendix B is filled from the scan targets and scan dates.

## Customer folders

For each scan, the script also builds the same folder that Parse-Nessus.ps1 produced on Prod, named like `Acme Health Inc External EIVA 2026 Q3`, next to the Word report. It contains `index.html`, the `reportsbyhost` and `reportsbyvuln` pages, `extras\hostinfo.csv`, `extras\ipsbyvuln.txt`, the At-A-Glance chart, and the `css` and `js` folders. It always includes Informational findings, as the old process did with VulnReportIncludeLevel 6, even when you skip them in the Word report.

On Windows the folder is byte for byte what the old script produced, because the script runs the original Parse-Nessus functions in Windows PowerShell 5.1. On Kali every page and CSV is also identical, except that `ipsbyvuln.txt` lists the same entries in a different order and the chart image is redrawn to look the same.

## Scan reports

A scan report fills in the cover, the findings tables, and the Appendix B scope with quarter dates such as Q3 2026. In the Vulnerability Plus/Delta Overview table, the script fills in the current quarter row with this scan's totals. The highlighted row above it is where you paste the client's history from their previous report, since that history is not in the .nessus file.

## Table of Contents page numbers

The script calculates the page numbers itself. The front pages never move, a scan report only grows when the findings tables pass 21 rows, and in an assessment each finding starts on its own page, with longer findings estimated to take extra pages. A finding with an unusually long description or asset list can occasionally push the numbers after it by a page, so glance at the Table of Contents and adjust if needed.

## Before you send the report

Open the report in Word, glance at the Table of Contents page numbers, then replace the client logo on the cover and check the Appendix B testing dates against the assignment. Save it as a PDF as usual.
