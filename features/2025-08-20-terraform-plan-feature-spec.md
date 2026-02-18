# Topic: Dry run deployment in Radius

* **Author**: 

## Topic Summary
<!-- A paragraph or two to summarize the topic area. Just define it in summary form so we all know what it is. -->
This feature introduces comprehensive dry run deployment functionality within Radius for both Bicep and Terraform infrastructure-as-code technologies. For Terraform, this leverages the terraform plan command to create execution plans, while for Bicep, this utilizes Azure Resource Manager's what-if operations to preview infrastructure changes before applying them, providing critical safety mechanisms for infrastructure management.

This feature enables platform engineers and application developers to preview, validate, and approve infrastructure changes through Radius-managed deployments using either Bicep or Terraform recipes. 

## Terms and definitions

- [Terraform Plan](https://developer.hashicorp.com/terraform/cli/commands/plan): A Terraform CLI command that creates an execution plan by comparing current infrastructure state with desired configuration, showing proposed changes without applying them.
- [ARM What-If](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-what-if): An Azure Resource Manager feature that previews changes that will happen when deploying Bicep templates or ARM templates without actually deploying them.

### Top level goals
<!-- At the most basic level, what are we trying to accomplish? -->
1. Enable users to preview infrastructure changes before applying both Bicep and Terraform recipes in Radius environments.
2. Provide comprehensive dry run output including change summaries, resource details for both infrastructure-as-code technologies.
4. Integrate dry run functionality with Radius CLI, API, and CI/CD automation scenarios for both Bicep and Terraform deployments.

### Non-goals (out of scope)
<!-- What are we explicitly not trying to accomplish? -->
1. Implement new drift detection mechanisms outside of Terraform's existing capabilities or ARM's what-if functionality.
2. Provide custom output formats; we will use Terraform's standard plan output and ARM's standard what-if result format.
3. Support dry run operations for infrastructure-as-code technologies other than Bicep and Terraform.

## User profile and challenges
<!-- Define the primary user and the key problem / pain point we intend to address for that user. If there are multiple users or primary and secondary users, call them out. -->

### User persona(s)
<!-- Who is the target user? Include size/org-structure/decision makers where applicable. -->

**Platform Engineer**: Responsible for building and maintaining the infrastructure platform that application teams use. They install and configure both Bicep and Terraform tooling, create Radius environments, manage infrastructure-as-code configuration, and develop Recipes using either technology.

**Application Developer**: Builds applications using Radius Resource types without direct knowledge of underlying Bicep or Terraform recipes. They declare what resources their application needs in Bicep files and deploy using `rad deploy`. Developers need to understand what infrastructure will be created for their application but shouldn't need to understand the underlying Recipe implementation details. They need dry run functionality to preview the infrastructure impact of their application deployments regardless of the Recipe technology used.

**DevOps Engineer**: Manages CI/CD pipelines and deployment automation for both platform engineering (recipe development) and application development workflows. They implement approval workflows, manage plan files and what-if results in automated deployments, and need to ensure infrastructure changes go through proper review processes before being applied for both Bicep and Terraform-based recipes.

**Site Reliability Engineer (SRE)**: Responsible for maintaining system reliability and managing infrastructure changes. They use dry run functionality to assess impact of changes, detect drift, and ensure infrastructure modifications align with operational requirements across both recipe-level and application-level deployments, regardless of whether recipes use Bicep or Terraform.

### Challenge(s) faced by the user
<!-- What challenges do the user face? Why are they experiencing pain and why do current offerings not meet their need? -->

**Cannot preview infrastructure changes**: Users lack the ability to see what infrastructure will be created, modified, or destroyed before applying Bicep or Terraform recipes in Radius. This creates risk of unexpected changes, cost implications, and operational issues regardless of the underlying infrastructure-as-code technology.

**No safety mechanism for deployments**: Teams cannot validate infrastructure changes before they're applied, leading to hesitation in adopting infrastructure-as-code recipes for critical infrastructure and limiting confidence in automated deployments across both Bicep and Terraform-based solutions.

**Difficult to debug recipe issues**: When Bicep or Terraform recipes fail or behave unexpectedly, users cannot preview what the recipe was trying to create, making troubleshooting and validation across environments challenging regardless of the Recipe implementation technology.

### Positive user outcome
<!-- What is the positive outcome for the user if we deliver this, i.e. what is the value proposition for the user? Remember, this is user-centric. -->

Users gain confidence and control over their infrastructure changes through Radius-managed deployments using both Bicep and Terraform. Platform engineers can design safer deployment workflows regardless of their chosen infrastructure-as-code technology, application developers can understand infrastructure impact before deployment without needing to know Recipe implementation details, and DevOps engineers can implement robust approval processes that work consistently across different Recipe technologies. The ability to preview, validate, and selectively apply changes reduces operational risk, improves change management processes, and enables more widespread adoption of Infrastructure as Code practices through Radius for both Azure-native (Bicep) and multi-cloud (Terraform) scenarios.

## Key scenarios
<!-- List ~3-7 high level scenarios to clarify the value and point to how we will decompose this big area into component capabilities. We may ultimately have more than one level of scenario. -->

**Scenario 1**: As an application developer or SRE, I want to preview the infrastructure changes that will be made by a Recipe (whether Bicep or Terraform) before applying it, so that I can understand the impact of the changes and approve or reject them based on the output.

**Scenario 2**: As a platform engineer, I want to validate that my Recipe changes (in either Bicep or Terraform) work correctly across different environments by running dry run operations before deploying to production.

**Scenario 3**: As a DevOps engineer, I want to integrate dry run capabilities into CI/CD pipelines for both Bicep and Terraform-based recipes to implement approval workflows and prevent unauthorized infrastructure changes.

## Detailed User Stories

### User Story 1: Application Developer Previewing Infrastructure Impact

**As an** application developer  
**I want to** preview the infrastructure changes that will be created by my application deployment before it gets applied  
**So that** I can understand the cost implications, security impact, and resource dependencies without needing to understand Recipe internals

#### Detailed User Experience:

**Context:** Sarah is an application developer working on a microservices application. She's about to deploy a new service that requires a database, message queue, and storage account. The platform team has created Recipes (some using Bicep, others using Terraform) for these resources, but Sarah doesn't know which technology each Recipe uses.

**Current Workflow:**
1. **Initiate Dry Run:** Sarah runs `rad deploy --dry-run app.bicep` from her application directory
2. **Recipe Discovery:** Radius identifies all the Recipe-backed resources in her application (Database, MessageQueue, StorageAccount)
3. **Technology-Agnostic Preview:** Radius executes the appropriate dry run operation for each Recipe:
   - Database Recipe (Terraform): Executes `terraform plan` behind the scenes
   - MessageQueue Recipe (Bicep): Executes ARM what-if operation
   - StorageAccount Recipe (Terraform): Executes `terraform plan`
4. **Unified Output Display:** Sarah sees a consolidated preview showing:
   ```
   Dry Run Results for Application: my-microservice
   
   Resources to be created:
   + Database (via Recipe: sql-database-recipe)
     - Azure SQL Database: my-app-db
     - Resource Group: rg-my-app-prod
     - Estimated monthly cost: $127.50
   
   + MessageQueue (via Recipe: servicebus-recipe) 
     - Service Bus Namespace: sb-my-app-prod
     - Service Bus Queue: orders-queue
     - Estimated monthly cost: $43.20
   
   + StorageAccount (via Recipe: blob-storage-recipe)
     - Storage Account: samyappprod001
     - Blob Container: documents
     - Estimated monthly cost: $15.80
   
   Total estimated monthly cost: $186.50
   Dependencies detected: Database depends on Resource Group
   ```

5. **Review and Decision:** Sarah reviews the output and notices:
   - The costs are within her team's budget
   - All resources will be created in the expected resource group
   - No unexpected dependencies or security groups are being created
   
6. **Proceed with Confidence:** Sarah runs `rad deploy app.bicep` knowing exactly what infrastructure will be created

**Key Benefits:**
- Sarah doesn't need to know whether Recipes use Bicep or Terraform
- She gets cost estimates before deployment
- She can validate that the right resources are being created in the right places
- She can catch potential issues (like naming conflicts) before they occur

### User Story 2: Platform Engineer Validating Recipe Changes Across Environments

**As a** platform engineer  
**I want to** validate Recipe changes in development before deploying to production environments  
**So that** I can ensure Recipe modifications work correctly across different environments and don't break existing deployments

#### Detailed User Experience:

**Context:** Mike is a platform engineer who maintains infrastructure Recipes for his organization. He's updating a Terraform-based database Recipe to add backup configuration and wants to ensure it works correctly before rolling it out to production teams.

**Current Workflow:**
1. **Recipe Development:** Mike modifies the database Recipe (written in Terraform) to include automated backup settings
2. **Development Environment Testing:** Mike runs `rad recipe dry-run database-recipe --environment dev` to test his changes:
   ```
   Recipe Dry Run: database-recipe (Terraform)
   Environment: dev
   
   Changes to be applied:
   ~ Azure SQL Database: dev-sample-db
     + backup_retention_days = 7
     + geo_redundant_backup_enabled = false
     + point_in_time_restore_enabled = true
   
   + SQL Database Backup Policy: dev-sample-db-backup
     + retention_days = 7
     + frequency = "Daily"
   ```

3. **Staging Environment Validation:** Mike tests the same Recipe in staging: `rad recipe dry-run database-recipe --environment staging`
   ```
   Recipe Dry Run: database-recipe (Terraform)
   Environment: staging
   
   Changes to be applied:
   ~ Azure SQL Database: staging-sample-db
     + backup_retention_days = 14
     + geo_redundant_backup_enabled = true
     + point_in_time_restore_enabled = true
   
   + SQL Database Backup Policy: staging-sample-db-backup
     + retention_days = 14
     + frequency = "Daily"
   ```

4. **Environment-Specific Validation:** Mike notices that staging automatically gets different backup settings (14 days vs 7 days retention, geo-redundant enabled) based on environment-specific parameters - exactly as intended

5. **Production Preview:** Before deploying, Mike runs `rad recipe dry-run database-recipe --environment prod` to see production impact:
   ```
   Recipe Dry Run: database-recipe (Terraform)
   Environment: prod
   
   Existing resources that will be modified:
   ~ Azure SQL Database: prod-orders-db (App: orders-api)
   ~ Azure SQL Database: prod-users-db (App: user-service)
   ~ Azure SQL Database: prod-inventory-db (App: inventory-service)
   
   Each database will receive:
   + backup_retention_days = 30
   + geo_redundant_backup_enabled = true
   + point_in_time_restore_enabled = true
   ```

6. **Risk Assessment:** Mike reviews the production preview and sees that 3 existing applications will be affected. He schedules the Recipe update during a maintenance window.

**Key Benefits:**
- Mike can validate Recipe behavior across multiple environments
- He can see the impact on existing applications before making changes
- Environment-specific configurations are properly tested
- He can coordinate Recipe updates with application teams affected

### User Story 3: DevOps Engineer Implementing Approval Workflows

**As a** DevOps engineer  
**I want to** integrate dry run results into CI/CD pipelines to require approval before infrastructure changes  
**So that** I can ensure all infrastructure modifications go through proper review processes

#### Detailed User Experience:

**Context:** Jennifer is a DevOps engineer setting up CI/CD pipelines for both application deployments and Recipe updates. She needs to ensure that any infrastructure changes are reviewed and approved before being applied.

**Current Workflow:**

**Pipeline for Application Deployments:**
1. **Automated Dry Run:** When a developer pushes changes to an application repository, the CI pipeline automatically runs:
   ```bash
   rad deploy --dry-run app.bicep --environment staging > dry-run-results.txt
   ```

2. **Pull Request Integration:** The pipeline posts dry run results as a comment on the pull request:
   ```
   🔍 Infrastructure Preview
   
   This deployment will create/modify the following resources:
   + Redis Cache: staging-app-cache (via redis-recipe)
   + Application Insights: staging-app-insights (via monitoring-recipe)
   
   Estimated cost impact: +$89.50/month
   
   ✅ Approve and merge to deploy to staging
   ```

3. **Staging Gate:** After merge, the pipeline deploys to staging and runs integration tests

4. **Production Approval:** For production deployment, the pipeline:
   - Runs `rad deploy --dry-run app.bicep --environment prod`
   - Creates a deployment request ticket with dry run results
   - Requires manual approval from both DevOps and Finance teams
   - Only deploys after approval

**Pipeline for Recipe Updates:**
1. **Recipe Validation:** When a platform engineer updates a Recipe:
   ```bash
   # Test Recipe across all environments
   rad recipe dry-run database-recipe --environment dev
   rad recipe dry-run database-recipe --environment staging  
   rad recipe dry-run database-recipe --environment prod
   ```

2. **Impact Analysis:** The pipeline generates an impact report:
   ```
   Recipe Update Impact Analysis
   Recipe: database-recipe (Terraform)
   
   Development: No existing resources affected
   Staging: 2 applications will be updated (app-a, app-b)
   Production: 7 applications will be updated (orders, users, inventory, etc.)
   
   Change Summary:
   + Automated backup configuration
   + Enhanced monitoring alerts
   
   Required Approvals: Platform Team Lead, Security Team
   ```

3. **Staged Rollout:** After approval, the pipeline:
   - Applies Recipe to dev environment
   - Runs validation tests
   - Applies to staging with monitoring
   - Waits 24 hours before production rollout
   - Applies to production during maintenance window

**Key Benefits:**
- Jennifer ensures no infrastructure changes bypass review processes
- Stakeholders can see cost and security impact before approval
- Recipe updates are carefully coordinated across environments
- Audit trail exists for all infrastructure modifications

### User Story 4: SRE Managing Infrastructure Drift and Changes

**As a** Site Reliability Engineer  
**I want to** detect and remediate infrastructure drift using dry run capabilities  
**So that** I can ensure production infrastructure matches its intended configuration and maintain system reliability

#### Detailed User Experience:

**Context:** Alex is an SRE responsible for maintaining production infrastructure for a critical e-commerce platform. They need to regularly check for drift and coordinate infrastructure updates while minimizing service disruption.

**Current Workflow:**

**Drift Detection Workflow:**
1. **Scheduled Drift Checks:** Alex sets up automated drift detection that runs weekly:
   ```bash
   # Check all Recipe-managed infrastructure for drift
   rad recipe drift-check --environment prod --all-recipes
   ```

2. **Drift Detection Results:** The system reports drift across different Recipe technologies:
   ```
   Infrastructure Drift Report - Production Environment
   Generated: 2024-03-15 09:00:00 UTC
   
   🔍 Database Recipe (Terraform):
   ~ Azure SQL Database: prod-orders-db
     - Current: backup_retention_days = 7
     + Expected: backup_retention_days = 30
     - Current: geo_redundant_backup_enabled = false  
     + Expected: geo_redundant_backup_enabled = true
   
   🔍 API Gateway Recipe (Bicep):
   ~ Application Gateway: prod-api-gateway
     - Current: sku_tier = "Standard_v2"
     + Expected: sku_tier = "WAF_v2"
     - Current: waf_enabled = false
     + Expected: waf_enabled = true
   
   🔍 Monitoring Recipe (Terraform):
   ✅ No drift detected
   
   Summary: 2 resources have drifted from expected configuration
   ```

3. **Risk Assessment:** Alex reviews each drift issue:
   - Database backup settings: Low risk, can be remediated immediately
   - API Gateway WAF: High risk, requires coordination with security team

**Coordinated Remediation Workflow:**
4. **Low-Risk Drift Remediation:** Alex immediately fixes the database backup drift:
   ```bash
   # Preview the remediation changes
   rad recipe dry-run database-recipe --environment prod --fix-drift
   ```
   
   ```
   Drift Remediation Preview: database-recipe (Terraform)
   
   Resources to be updated:
   ~ Azure SQL Database: prod-orders-db
     + backup_retention_days = 30
     + geo_redundant_backup_enabled = true
   
   Estimated downtime: None
   Risk level: Low
   ```

5. **High-Risk Change Coordination:** For the API Gateway WAF change, Alex coordinates with multiple teams:
   ```bash
   # Generate detailed impact analysis
   rad recipe dry-run api-gateway-recipe --environment prod --fix-drift --detailed
   ```
   
   ```
   High-Impact Change Analysis: api-gateway-recipe (Bicep)
   
   Resources to be updated:
   ~ Application Gateway: prod-api-gateway
     + sku_tier = "WAF_v2"
     + waf_enabled = true
   
   Applications affected: 12 services routing through this gateway
   Estimated impact: 5-10 minutes service interruption during update
   Dependencies: External DNS, SSL certificates
   
   Recommended maintenance window: Saturday 02:00-04:00 UTC
   Required approvals: Security Team, Platform Lead, On-call Engineer
   ```

6. **Maintenance Window Execution:** During the planned maintenance window:
   ```bash
   # Final verification before applying changes
   rad recipe dry-run api-gateway-recipe --environment prod --fix-drift
   
   # Apply the drift remediation
   rad recipe apply api-gateway-recipe --environment prod --fix-drift
   ```

**Emergency Change Workflow:**
7. **Security-Critical Drift:** When Alex discovers a security-critical configuration drift:
   ```
   🚨 Critical Drift Detected: storage-recipe (Bicep)
   ~ Storage Account: prodappstorage001
     - Current: public_access_enabled = true
     + Expected: public_access_enabled = false
   
   Security Impact: HIGH - Storage account has public access enabled
   Immediate action required
   ```

8. **Emergency Remediation:** Alex can quickly preview and apply the security fix:
   ```bash
   # Emergency dry run with security focus
   rad recipe dry-run storage-recipe --environment prod --security-critical
   
   # Apply immediately after stakeholder notification
   rad recipe apply storage-recipe --environment prod --security-critical
   ```

**Key Benefits:**
- Alex can proactively detect configuration drift before it impacts services
- Different remediation workflows for different risk levels
- Clear impact analysis helps coordinate changes with other teams
- Emergency procedures for security-critical drift
- Consistent approach works across both Bicep and Terraform-based recipes
- Comprehensive audit trail for compliance requirements

## Core Workflow

### Dry Run Process for Both Technologies

1. **Preview infrastructure changes before deployment**
   - For Terraform recipes: Use `terraform plan` to show what resources will be created, modified, or destroyed
   - For Bicep recipes: Use ARM what-if operations to preview changes to Azure resources
   - Display consistent output format regardless of underlying technology

2. **Generate preview artifacts**
   - For Terraform: Generate plan files with modes including normal, destroy, and refresh-only
   - For Bicep: Generate what-if results with incremental and complete deployment modes
   - Store results for later review and application

3. **Act on preview results**
   - Approve or reject changes based on dry run output
   - Apply approved changes using the generated plan/what-if results
   - Reconcile drift by applying changes identified in the preview

### Technology-Specific Considerations

**Terraform Integration:**
- Leverage existing Terraform CLI commands and plan file format
- Support all Terraform planning modes and configurations
- Maintain compatibility with existing TerraformSettings resources

**Bicep Integration:**
- Utilize Azure Resource Manager what-if API for preview operations
- Support both template-level and resource group-level what-if operations
- Integrate with existing BicepSettings and Azure provider configurations

**Unified Experience:**
- Provide consistent CLI commands and API interfaces regardless of Recipe technology
- Abstract technology-specific details while preserving full functionality
- Enable users to work with dry run operations without needing to understand underlying implementation differences 







