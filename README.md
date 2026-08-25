# Power Platform Enterprise ALM Pipelines for Azure DevOps

A modular, production-ready Azure DevOps YAML pipeline framework for Microsoft Power Platform solutions and Power Pages (Enhanced Data Model).

Designed with industry best practices, Microsoft Power Platform Build Tools, automated build versioning (`YYYYMMDD.rev`), Solution Checker analysis, source unpacking, and multi-stage gated deployments (Test, Staging, Production) with deployment settings files.

---

## 🌟 Key Features

1. **Modular Architecture**: Clean separation between root orchestrator pipelines, stage templates, and reusable task step templates.
2. **Build-based Versioning**: Automatic calculation and injection of `YYYYMMDD.rev` (e.g. `20260825.1`) into the Dataverse solution prior to export.
3. **Power Platform Build Tools**: Built on official Microsoft Azure DevOps tasks (`PowerPlatformToolInstaller`, `PowerPlatformPublishCustomizations`, `PowerPlatformSetSolutionVersion`, `PowerPlatformExportSolution`, `PowerPlatformUnpackSolution`, `PowerPlatformChecker`, `PowerPlatformImportSolution`).
4. **Power Pages Support (Enhanced Data Model)**: Works out of the box because website components are packaged natively inside the Dataverse solution.
5. **Quality & Governance Gates**: Automated Power Platform Solution Checker step with configurable rule-level thresholds (warn or fail on critical issues) before publishing build artifacts.
6. **Git Source Sync & Artifact Publishing**: Unpacks unmanaged solution XML/JSON to source layout and publishes managed zip artifacts to pipeline drop.
7. **Multi-Environment Deployment**: Multi-stage release using Azure DevOps Environments (allowing manual approval checks, business approvals, and audit trails) with environment-specific `DeploymentSettings.json` mappings.

---

## 📁 Repository Structure

```
├── .gitignore
├── README.md
├── pipelines/
│   ├── build-and-release-solution.yml       # Main root orchestrator pipeline
│   └── variables/
│       ├── common-vars.yml                  # Global solution & build configuration
│       ├── env-test-vars.yml                # Test environment variables
│       ├── env-staging-vars.yml             # Staging environment variables
│       └── env-prod-vars.yml                # Production environment variables
├── templates/
│   ├── stages/
│   │   ├── stage-build-and-export.yml       # Dev export, version bump, unpack & checker
│   │   └── stage-deploy-environment.yml     # Reusable deployment stage for downstream envs
│   └── steps/
│       ├── step-tool-installer.yml          # Tool installer & PowerShell prerequisites
│       ├── step-version-solution.yml        # Calculate & apply YYYYMMDD.rev version
│       ├── step-publish-customizations.yml  # Publish all customizations in Dev
│       ├── step-export-solutions.yml        # Export Managed & Unmanaged solution zips
│       ├── step-unpack-solution.yml         # Unpack unmanaged solution to repo structure
│       ├── step-solution-checker.yml        # Run Power Apps Solution Checker
│       ├── step-import-solution.yml         # Import managed solution with deployment settings
│       └── step-publish-artifacts.yml       # Publish artifacts (managed zip, checker report)
└── config/
    └── deployment-settings/
        ├── test-deployment-settings.json    # Sample Test deployment settings (env vars & conn refs)
        ├── staging-deployment-settings.json # Sample Staging deployment settings
        └── prod-deployment-settings.json    # Sample Prod deployment settings
```

---

## 🚀 Getting Started & Setup Guide

### 1. Prerequisites in Azure DevOps
1. **Install Extension**: Install the [Power Platform Build Tools](https://marketplace.visualstudio.com/items?itemName=microsoft-IsvExpPlatform.PowerPlatform-BuildTools) extension in your Azure DevOps organization.
2. **Service Connections**: Create Generic Power Platform Service Connections (recommended: Service Principal with Client Secret or Workload Identity Federation):
   - `PowerPlatform-Dev`
   - `PowerPlatform-Test`
   - `PowerPlatform-Staging`
   - `PowerPlatform-Prod`
3. **Environments & Approvals**: In Azure DevOps (*Pipelines* -> *Environments*), create:
   - `PP-Test` (Optionally add approval checks)
   - `PP-Staging` (Configure pre-deployment approval gates)
   - `PP-Production` (Configure pre-deployment approval gates & change windows)

### 2. Configure Solution & Variables
Edit `pipelines/variables/common-vars.yml`:
- `solutionName`: The unique schema name of your Dataverse solution (e.g. `ContosoEnterpriseCore`).
- `majorMinorPrefix`: Set base version prefix if desired (or use standard `YYYYMMDD.rev`).

Edit the deployment settings in `config/deployment-settings/` to map:
- **Connection References**: Map connection reference IDs to target connection IDs.
- **Environment Variables**: Configure values per environment (e.g. API endpoints, tenant IDs, feature flags).

---

## 🔒 Security Best Practices
- Never commit real client/tenant credentials to Git. Use Azure DevOps Variable Groups or Azure Key Vault task integration for sensitive secrets.
- Use generic placeholders for solution names and schema elements.
- Apply branch protection rules to `main` so that all releases follow gated PRs and pipeline runs.
