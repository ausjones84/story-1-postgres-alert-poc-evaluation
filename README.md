# Story 1 — Evaluate and Test PostgreSQL Flexible Server Alert POC

![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-0089D6?style=for-the-badge&logo=microsoftazure&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![Status](https://img.shields.io/badge/Status-In_Progress-yellow?style=for-the-badge)

## Ticket Reference
| Field | Value |
|---|---|
| **Story** | Story 1 |
| **Title** | Evaluate and test PostgreSQL Flexible Server alert POC script |
| **Sprint** | EDAV PostgreSQL Alert Automation |
| **Assignee** | Austin Jones |
| **Reviewer** | Krishna |

---

## Purpose

This repository contains the evaluation and findings for the PostgreSQL Flexible Server alert POC script provided by Krishna. The goal is to validate the current Terraform structure, identify module gaps, attempt local validation, and document all findings before integrating into the standardized enterprise Terraform alert framework.

---

## What This Story Covers

- Review the folder structure of the Postgres alert POC
- - Validate DEV and PRD environment Terraform scripts
  - - Document all alert types defined in the POC
    - - Identify missing or broken module references
      - - Run `terraform init` with backend disabled to catch dependency errors
        - - Record all findings for the team and ticket update
         
          - ---

          ## POC Folder Structure Reviewed

          ```
          postgres-alerts/
          ├── modules/
          │   ├── actionGroup/
          │   └── dataCollectionRule/
          └── environments/
              ├── dev/postgres-alerts/
              │   └── metricAlert.tf
              └── prd/postgres-alerts/
                  └── metricAlert.tf
          ```

          ---

          ## Alerts Found in the POC

          | Alert Name | Metric | Operator | Threshold | Severity |
          |---|---|---|---|---|
          | CPU High Critical | cpu_percent | GreaterThan | 85 | 1 |
          | CPU High Warning | cpu_percent | GreaterThan | 70 | 2 |
          | Memory High Critical | memory_percent | GreaterThan | 85 | 1 |
          | Memory High Warning | memory_percent | GreaterThan | 70 | 2 |
          | Disk High Critical | storage_percent | GreaterThan | 85 | 1 |
          | Active Connections | active_connections | GreaterThan | 343 | 2 |
          | Failed Connections | connections_failed | GreaterThan | 5 | 1 |

          ---

          ## Key Finding — Missing Module

          The DEV and PRD environment scripts reference a module path that was **not included** in the POC package:

          ```hcl
          source = "../../../modules/metric_alerts"
          ```

          This module (`modules/metric_alerts`) does not exist in the uploaded folder. This is the primary blocker for running `terraform init` successfully.

          **Resolution path:** Create a standardized `modules/global_metric_alert` module (Story 2) to replace this missing dependency.

          ---

          ## How to Run Locally

          ### Prerequisites
          - [Terraform >= 1.5](https://developer.hashicorp.com/terraform/downloads)
          - - Azure CLI authenticated (`az login`)
            - - Access to the EDAV DEV subscription
             
              - ### Steps
             
              - ```bash
                # 1. Clone this repo
                git clone https://github.com/ausjones84/story-1-postgres-alert-poc-evaluation.git
                cd story-1-postgres-alert-poc-evaluation

                # 2. Navigate to the DEV environment folder
                cd environments/dev/postgres-alerts

                # 3. Attempt init with backend disabled (for local validation only)
                terraform init -backend=false

                # 4. Expected result if module is missing:
                # Error: Unreadable module directory
                # ../../../modules/metric_alerts does not exist

                # 5. Run validate after init succeeds
                terraform validate
                ```

                ### Expected Output (Before Fix)
                ```
                Error: Unreadable module directory
                  on metricAlert.tf line 3, in module "postgres_cpu_alert_85":
                  source = "../../../modules/metric_alerts"
                The directory ../../../modules/metric_alerts does not exist.
                ```

                This output **is your Story 1 finding**. Document it and share with Krishna.

                ---

                ## Ticket Update (Copy-Paste Ready)

                ```
                Reviewed the PostgreSQL alert POC provided by Krishna. Confirmed the POC includes DEV and PRD
                alert definitions for CPU (>85, >70), memory (>85, >70), storage (>85), active connections (>343),
                and failed connections (>5) for PostgreSQL Flexible Servers.

                Key finding: The environment scripts reference ../../../modules/metric_alerts, but this module
                was not included in the provided POC package. Terraform init with backend disabled fails with
                "Unreadable module directory" error.

                Next step: Story 2 will create the standardized global_metric_alert module to resolve this
                dependency and enable full validation.
                ```

                ---

                ## Architecture Flow

                ```
                PostgreSQL POC (this repo)
                        |
                        v
                modules/metric_alerts  <-- MISSING (Story 2 will create this)
                        |
                        v
                Azure Monitor Metric Alert
                        |
                        v
                Action Group --> PagerDuty / Email Notification
                ```

                ---

                ## Repository Structure (This Repo)

                ```
                story-1-postgres-alert-poc-evaluation/
                ├── environments/
                │   ├── dev/postgres-alerts/
                │   │   └── metricAlert.tf        # DEV alert definitions (from POC)
                │   └── prd/postgres-alerts/
                │       └── metricAlert.tf        # PRD alert definitions (from POC)
                ├── modules/
                │   ├── actionGroup/              # Action group module (from POC)
                │   └── dataCollectionRule/       # DCR module (from POC)
                ├── docs/
                │   ├── poc-review-findings.md    # Full findings document
                │   └── alert-inventory.md        # All alert types documented
                ├── .gitignore
                └── README.md
                ```

                ---

                ## Open Questions for Krishna

                - [ ] Where is the `modules/metric_alerts` module — does it live in another repo?
                - [ ] - [ ] Should the new module be named `global_metric_alert` for consistency with VM alerts?
                - [ ] - [ ] Should failed connections use metric alerts or query-based (KQL) alerts?
                - [ ] - [ ] What action group ARN/ID should be used per environment (DEV/PRD/DMZ)?
                - [ ] - [ ] Should alert creation be part of the PostgreSQL provisioning pipeline or a standalone alert pipeline?
               
                - [ ] ---
               
                - [ ] ## Enhancement Notes (Pro Tips)
               
                - [ ] - Run `terraform fmt -check` to enforce formatting before commit
                - [ ] - Add `terraform validate` as a pre-commit hook using `pre-commit` framework
                - [ ] - Use `checkov` for static security analysis of Terraform modules
                - [ ] - All module inputs should use `sensitive = true` where applicable for secrets
               
                - [ ] ---
               
                - [ ] ## Related Stories
               
                - [ ] | Story | Repo | Description |
                - [ ] |---|---|---|
                - [ ] | Story 1 | **This repo** | POC evaluation and findings |
                - [ ] | Story 2 | [story-2-global-metric-alert-module](https://github.com/ausjones84/story-2-global-metric-alert-module) | Create reusable metric alert module |
                - [ ] | Story 3 | [story-3-global-query-alert-module](https://github.com/ausjones84/story-3-global-query-alert-module) | Create reusable query alert module |
                - [ ] | Story 4 | [story-4-postgres-cpu-alert-integration](https://github.com/ausjones84/story-4-postgres-cpu-alert-integration) | CPU alert provisioning |
                - [ ] | Story 5 | [story-5-postgres-memory-alert-integration](https://github.com/ausjones84/story-5-postgres-memory-alert-integration) | Memory alert provisioning |
                - [ ] | Story 6 | [story-6-postgres-disk-alert-integration](https://github.com/ausjones84/story-6-postgres-disk-alert-integration) | Disk alert provisioning |
                - [ ] | Story 7 | [story-7-postgres-connection-alert-integration](https://github.com/ausjones84/story-7-postgres-connection-alert-integration) | Connection alert provisioning |
                - [ ] | Story 8 | [story-8-postgres-restore-alert-rules](https://github.com/ausjones84/story-8-postgres-restore-alert-rules) | Restore alert rules from backup |
