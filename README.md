# JF Ignore Rules Management Script
 This code is an example to manage JF Ignore Rules with a GitHub workflow, and an .jfignore file.

## Script Overview


#### The provided Bash script is designed to manage ignore rules for CVEs (Common Vulnerabilities and Exposures) in JF. 

#### It performs the following tasks:

- [x] Checks the provided arguments.

- [x] Reads ignore rules from a specified file.

- [x] Adds or replaces ignore rules in JF using JF CLI.

### Script Usage

To use the script, you need to pass three parameters:
```bash
./ir <project_name> <watch> <ignore_file>
```

`<project_name>`: The JF project name.

`<watch>`: The watch parameter to filter the rules.

`<ignore_file>`: The path to the file containing the ignore rules.


### Key Functions

1. `format_expiration`

Formats the expiration date into ISO 8601 format.

2. `create_curl_request`

Creates a new cURL request to add or replace an ignore rule.

3. `get_existing_ignore_rules`

Fetches the current ignore rules from JF.

4. `check_and_create_rule`

Checks if a rule already exists and either replaces or adds it accordingly.


## GitHub Workflow

The GitHub workflow automates the process of applying ignore rules when changes are pushed to the repository.

#### Workflow YAML
```yaml
name: "JF Ignore Rules Trigger"

on:
  push:
    paths:
      - 'ignore/.jfignore'

env:
  JFPROJECT: ${{ vars.JFPJ }}
  WATCHES: ${{ vars.WHATCH }}
  PATHIGNORERULE: ${{ vars.IR }}

jobs:
  build:
    name: "Ignore CVE"
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - uses: jfrog/setup-jfrog-cli@v4
      env:
        JF_URL: ${{ secrets.ARTIFACTORY_URL }}
        JF_USER: ${{ secrets.ARTIFACTORY_USERNAME }}
        JF_PASSWORD: ${{ secrets.ARTIFACTORY_PASSWORD }}

    - name: Make script executable
      run: chmod +x ./scripts/ir

    - name: Add or replace ignore rules
      run: |
        ./scripts/ir $JFPROJECT $WATCHES $PATHIGNORERULE
```


#### Workflow Steps

- [x] Checkout Repository: Pulls the latest code from the repository.

- [x] Setup JF CLI: Sets up the JF CLI with necessary credentials.

- [x] Make Script Executable: Ensures the script is executable.

- [x] Execute Script: Runs the script with the specified parameters.


## `.jfignore` File

The `.jfignore` file lists the CVEs to be ignored along with their optional expiration dates.

### Format

Each line contains a CVE ID and an optional expiration date in the format exp:YYYY-MM-DD.

## Example

```
CVE-2022-22965 exp:2025-01-23
CVE-2024-159 exp:2025-01-24
CVE-2024-170 exp:2025-01-24
CVE-2024-171
CVE-2024-172
```

`CVE-2022-22965 exp:2025-01-23`: This rule will expire on January 23, 2025.

`CVE-2024-171`: This rule has no expiration date.

### Rules management
The script manages ignore rules based on the comparison between the `.jfignore` file and existing rules in JF.

| **Scenario**                                                                                     | **Action**                                                                                              |
|---------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| **In .jfignore:** CVE with expiration date **In JF:** No CVE, no expiration date                  | Create the ignore rule in JF.                                                                           |
| **In .jfignore:** CVE with no expiration date **In JF:** No CVE, no expiration date               | Create the ignore rule in JF.                                                                           |
| **In .jfignore:** CVE with expiration date **In JF:** CVE with no expiration date                 | Create a new ignore rule in JF with the expiration date and delete the one without expiration date.      |
| **In .jfignore:** CVE with no expiration date **In JF:** CVE with no expiration date              | No action needed, the rule already exists.                                                              |
| **In .jfignore:** CVE with expiration date **In JF:** CVE with the same expiration date           | No action needed, the rule already exists.                                                              |
| **In .jfignore:** CVE with expiration date **In JF:** CVE with an earlier expiration date         | Create a new ignore rule in JF with the updated expiration date and keep the existing rule.              |
| **In .jfignore:** CVE with expiration date **In JF:** CVE with a later expiration date            | Create a new ignore rule in JF with the updated expiration date and delete the rules with a later date.  |


## Summary

This guide provides the necessary details to understand and use the JF Ignore Rules Management Script, configure the GitHub workflow, and properly format the .jfignore file for managing CVE ignore rules in JF.
