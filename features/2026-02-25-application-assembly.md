# Topic: Application Assembly

* **Author**: Reshma Abdul Rahim (@reshrahim)

## Topic Summary

Enterprise developers today face a fundamental disconnect between writing application code via AI coding assistants and deploying it in compliance with organizational platform standards. Platform engineers spend significant time defining infrastructure policies, approved modules, and deployment conventions, but developers have no easy way to discover or apply these standards. The result is a slow, manual, and error-prone process where compliance is enforced reactively instead of being built in from the start.

Application Assembly is a new layer in Radius that automatically discovers application components from existing code artifacts, infers platform requirements based on organizational standards, and generates a deployable application definition that is compliant by default.

### Principles

1. **Developer-first**: Optimize for a seamless developer experience, while honoring platform engineering standards. The default path should be simple for application teams and have basic compliance with organizational policies.

2. **Cognitive load reduction**: Use developer language and hide internal Radius concepts (Resource Types, Recipes, Bicep) by default. Users should get value without learning the internal Radius resource model.

3. **Progressive adoption**: Generate a working starting point that is easy to extend and customize as users learn more. The initial experience is low-friction, with a clear path to deeper control.

4. **Trust and transparency**: Always show what was inferred and from where. Require review before deploy, and make it easy to accept, edit, or reject each inference.

### Goals

1. Introduce an application-intent layer that allows developers to create new applications that are compliant with organizational standards.
2. Reduce onboarding friction for users and accelerate time to value by enabling an experience that automatically discovers application components, infers platform requirements, and generates a deployable Radius application definition.
3. Provide a deterministic and transparent application assembly process that developers can trust and understand.

### Non-goals (out of scope)

1. Radius does not guide how the developer builds the application source code or how they use AI coding assistants to scaffold and iterate on their application.

## User profile and challenges

Application Assembly targets two primary personas who collaborate across the platform engineering workflow: **enterprise application developers** who build and ship applications, and **platform engineers** who define and enforce organizational infrastructure standards.

### User persona(s)

**Enterprise Application Developer**

Application developers in enterprises who are building new applications and want to deploy them in compliance with organizational standards.

- **Established developers**: Experienced engineers bootstrapping new services who want to move fast without manually researching which infrastructure modules are approved, how to wire up dependencies and how to handle secrets. They want to focus on their application code and let the platform handle the rest.

- **New developers**: Engineers from adjacent disciplines (data analysts, scientists, designers) or new hires who need to stand up compliant applications but lack familiarity with the organization's infrastructure tooling and policies.

Both subgroups use AI coding assistants or other tools as part of their daily workflow and expect tooling to meet them where they are rather than forcing them to learn new languages or paradigms upfront.

**Platform Engineer**

Platform engineers who manage the infrastructure and define organizational standards with approved cloud providers, compute targets, IaC modules, naming conventions, compliance requirements, and secret management policies. They need a way to codify these standards and provide it to the developer tooling so that developer tooling can automatically enforce them before deployment.

### Challenge(s) faced by the user

**For application developers:**

1. **Infrastructure complexity**: Developers are good at application code, not infrastructure. Deploying a compliant app means learning IaC tools like Terraform, Bicep, or Helm, cloud resource models, and internal platform abstractions. This adds cognitive overhead, slows development, and forces context switching between writing app logic and managing infrastructure config.

2. **Platform standards are fragmented**: Organizational standards live in wikis, scattered docs, or sometimes just tribal knowledge. Developers don't know where to start, which leads to repeated back and forth with the platform team and slows delivery.

3. **Compliance is discovered too late**: Developers make infrastructure decisions early like database images, caches, or auth methods without any guardrails. Compliance issues get caught later in code review, CI/CD checks, or even after deployment. That means rework, delayed releases, and frustration for both teams.

4. **AI coding assistants lack platform context**: Developers use AI tools to scaffold and build apps, but these tools don't know the org's platform standards, approved modules, or compliance rules. The code might work, but it doesn't match what the platform team expects, creating drift and new compliance problems.

**For platform engineers:**

1. **Platform standards are enforced manually**: Platform engineers define standards but do not have an easy and automated way to enforce them across organizations. This leads to PEs spending time fixing misconfigurations instead of improving the platform.

### Positive user outcome

- Developers can use AI coding assistants without worrying about generating non-compliant code.

- Developers can go from code to a deployable, compliant application without learning infrastructure tools or platform internals.

- Developers can see what the system inferred and why, and accept, edit, or reject each inference before deployment.

- Platform engineers codify standards in a Platform Constitution that is automatically applied during application assembly. Updates propagate to new applications automatically.

- Compliance is enforced before deployment instead of after.

### Scenarios

#### Scenario 1 : As an enterprise application developer, I want to create a new application using copilot that match the enterprise platform standards

A developer is creating a new application using Copilot CLI. They want to ensure that the application is compliant with the enterprise platform standards defined by the platform engineering team, such as using approved Terraform modules, following naming conventions, and adhering to authentication preferences.

#### Scenario 2: As a platform engineer, I want to define and maintain organizational standards in a single place that can be automatically applied during application assembly

A platform engineer wants to codify the organization's infrastructure standards, such as approved cloud providers, compute platforms, IaC modules, naming conventions, and compliance requirements in a Platform Constitution. This constitution should be easily maintainable and automatically applied during application assembly to ensure all applications adhere to these standards.

#### Scenario 3: As an enterprise application developer, I want to update my existing application to be compliant with the latest platform standards without having to manually refactor my codebase

A developer has an existing application that was built in a non-compliant manner. They want to update their application to be compliant with the latest standards without having to manually refactor their codebase.

### End-to-End User Journey

**Scenario** : Maria is a developer at MyCompany.Inc. She is an experienced developer but new to the company's platform standards. She wants to build a new application that builds customer feedback service and deploy it using Copilot CLI while ensuring compliance with the platform standards.

**Pre-requisites** - Maria has Copilot CLI installed.

```
╭──────────────────────────────────────────────────────────────╮
│                                                              │
│  ◻◻  GitHub Copilot v0.0.422-0                               │
│  ▮▮▮ Describe a task to get started.                         │
│                                                              │
│  Tip: /instructions View and toggle custom instruction files │
│  Copilot uses AI, so always check for mistakes.              │
│                                                              │
╰──────────────────────────────────────────────────────────────╯
```

```
> mkdir customer-feedback-service
> cd customer-feedback-service
```

#### Install Radius skills

Maria installs Radius skills in her repository based on the instructions provided by the platform team so that she can build compliant applications from day one.

```
❯ npx add radius-skills 

● Add skills from radius-project/skills repo

● The repo cloned successfully and found 3 skills. It's waiting for you to select which skills to install. Which skills would you like? You can choose from categories like:

   - Platform constitution – Organize the platform standards and policies that govern how applications should be built and deployed within your organization.
   - Application Modelling – Automatically scaffold a compliant application definition based on the developer's intent and platform constitution.
   - Application Verification – Check the compliance of an application definition against the platform constitution and certify the application for deploymnent

● Configured Radius MCP server in .github/skills/mcp-server.js 

● All 3 skills installed successfully to .github/skills/ in your project. MCP server will start automatically when coding agent is launched.
```

>[!NOTE]
> The Radius skills and the MCP server are packaged together and available as a single installation. We can also add it to other package managers like pip, brew, etc or via rad CLI as well.

#### Building the application source code with Copilot

>[!IMPORTANT]
> Copilot builds the application source code based on Maria's input. There is no Radius skills involvement in this step and is left to the user to guide copilot in building the app as they normally would.

Now Maria is ready to build the customer feedback service application.

```
❯ build a customer feedback service

◐ This is a fairly empty repo with just a Radius MCP server. The user wants to build a customer feedback service. I need to clarify what they want - tech stack, features, etc. Also, there's no Platform Engineering Constitution, which my instructions say I should generate first for infrastructure-related tasks.

  Let me ask the user some key questions about what they want.

● I see this is a repo with a Radius MCP server setup. Before building the customer feedback service, I have a few questions to scope it properly.

○ Asking user:  What tech stack would you like for the customer feedback ...

╭──────────────────────────────────────────────────────────╮
│ What tech stack would you like for the customer          │
│ feedback service?                                        │
│                                                          │
│ ❯ 1. Node.js/Express with TypeScript (Recommended)       │
│   2. Python/FastAPI                                      │
│   3. Go                                                  │
│   4. Other (type your answer)                            │
│                                                          │
│ ↑↓ to select · Enter to confirm · Esc to cancel          │
╰──────────────────────────────────────────────────────────╯
● Asked user: What tech stack would you like for the customer feedback ...
  └ User selected: Python/FastAPI

● Asked user: What features do you need in the feedback service?
  └ User selected: Full CRUD: submit, list, get, update, delete feedback with rat...

● Asked user: What database would you like to use for storing feedback?
  └ User selected: PostgreSQL

The customer feedback service is built and validated. Here's the summary:

feedback-service/ — Python/FastAPI + PostgreSQL with full CRUD:

┌───────────────────────────┬─────────────────────────────────────────────────────┐
│ Endpoint                  │ Description                                         │
├───────────────────────────┼─────────────────────────────────────────────────────┤
│ POST /api/v1/feedback/    │ Submit feedback (name, email, category, rating 1-5) │
│ GET /api/v1/feedback/     │ List with pagination + filter by category/rating    │
│ GET /api/v1/feedback/{id} │ Get by ID                                           │
│ PUT /api/v1/feedback/{id} │ Partial update                                      │
│ DELETE /api/v1/feedback/  │ Delete                                              │
└───────────────────────────┴─────────────────────────────────────────────────────┘
```

#### Building the application definition and deploying the application

**Platform Constitution Generation**

This step is independent of the Radius application assembly layer. The Platform Constitution serves as an input to application assembly but can also be consumed by other tools outside Radius.

In the developer flow, the `platform-constitution` skill produces a lightweight constitution sufficient for application modeling. A more robust platform constitution experience can be provided later on after we have more feedback from the users.

```
> build and deploy the application

◐ No Platform-Engineering-Constitution.md found.
  Need to generate one before proceeding with deployment.

○ Asking user: What do you mean by "build and deploy"?

╭──────────────────────────────────────────────────────────╮
│ The repo has a feedback-service (Python/FastAPI +        │
│ PostgreSQL) and an mcp-server (Node.js). What do you     │
│ mean by "build and deploy"?                              │
│                                                          │
│ ❯ 1. Run locally with Docker Compose (Recommended)       │
│   2. Deploy to cloud (requires Platform Constitution)    │
│   3. Just build the Docker images without running        │
│   4. Other (type your answer)                            │
│                                                          │
│ ↑↓ to select · Enter to confirm · Esc to cancel          │
╰──────────────────────────────────────────────────────────╯
```

> **Open Question**: Copilot generated source code has docker-compose.yml. How much of a support is required for local dev and testing for enterprise developers? Should Radius skills jump to generating the `app.bicep` and let users run it in the cloud instead of running locally with docker-compose?

```
◐ No Platform-Engineering-Constitution.md found.
  Invoking platform-constitution skill...

● skill(platform-constitution)

○ Asking user: Do you have existing infrastructure docs?
╭──────────────────────────────────────────────────────────╮
│ Do you have existing documentation about your            │
│ infrastructure practices? (e.g., Confluence, Notion,     │
│ ADRs, runbooks, policy docs)                             │
│                                                          │
│   1. No — let's start from scratch                       │
│ > 2. Other (type your answer)                            │
│                                                          │
│ ↑↓ to select · Enter to confirm · Esc to cancel          │
╰──────────────────────────────────────────────────────────╯
● Asked user: Do you have existing infrastructure docs?
  └ User entered: https://mycompany.inc/wiki/platform-guide

◐ Fetching https://mycompany.inc/wiki/platform-guide...
  └ Retrieved 12 pages (platform guide, ADRs, runbooks)

◐ Analyzing documentation for platform standards...

  Found the following from your docs:
    ✔ Cloud provider: Azure
    ✔ Compute platform: Kubernetes (AKS)
    ✔ IaC tool: Terraform
    ✔ Registry: mycompany.azurecr.io
    ✖ Naming conventions: not found
    ✖ Compliance / tagging requirements: not found

  I was able to extract some standards from your docs,
  but a few areas are missing. Let me ask you about those.

○ Asking user: Naming conventions

╭──────────────────────────────────────────────────────────╮
│ What naming convention does your org follow for          │
│ cloud resources?                                         │
│                                                          │
│ ❯ 1. {company}-{env}-{app}-{resource}                    │
│   2. {team}-{app}-{env}-{resource}                       │
│   3. We don't have a convention yet                      │
│   4. Other (type your answer)                            │
│                                                          │
│ ↑↓ to select · Enter to confirm · Esc to cancel          │
╰──────────────────────────────────────────────────────────╯
● Asked user: Naming conventions
  └ User selected: {company}-{env}-{app}-{resource}
● Asked user: What is the company prefix?
  └ User entered: mycompany

○ Asking user: Required tags

╭──────────────────────────────────────────────────────────╮
│ Which tags are required on all cloud resources?          │
│ (select all that apply)                                  │
│                                                          │
│ ❯ [x] environment                                        │
│   [x] team                                               │
│   [x] cost-center                                        │
│   [ ] data-classification                                │
│   [ ] Other (type your answer)                           │
│                                                          │
│ ↑↓ to select · Space to toggle · Enter to confirm        │
╰──────────────────────────────────────────────────────────╯
● Asked user: Required tags
  └ User selected: environment, team, cost-center

◐ Generating Platform-Engineering-Constitution.md...

  Created: Platform-Engineering-Constitution.md

  ┌─────────────────────────────────────────────────────────┐
  │ Section                       │ Source                  │
  ├─────────────────────────────────────────────────────────┤
  │ 1. Organization Overview      │ wiki (extracted)        │
  │ 2. Cloud Providers: Azure     │ wiki (extracted)        │
  │ 3. Compute: Kubernetes (AKS)  │ wiki (extracted)        │
  │ 4. IaC Tool: Terraform        │ wiki (extracted)        │
  │ 5. Module Catalog (14 modules)│ registry (fetched)      │
  │ 6. Naming: mycompany-{env}-…  │ questionnaire           │
  │ 7. Tags: env, team, cost-ctr  │ questionnaire           │
  └─────────────────────────────────────────────────────────┘

  ℹ Sources: 4 sections from docs, 1 from registry,
    2 from questionnaire

? Review the constitution before proceeding? (Y/n): Y

◐ Opening Platform-Engineering-Constitution.md in editor...

? Does the constitution look correct? (Y/n): Y

✔ Platform Engineering Constitution saved
```

An example of the generated `Platform-Engineering-Constitution.md` is included in the Appendix section below.

> **Open Questions**
> 1. Can a tool be added to the MCP server to authenticate to private docs which allows Copilot to crawl and fetch the platform data?
> 2. What are the authentication mechanisms we can support for private docs? (e.g., OAuth, PAT, SSO)
> 3. If we cannot authenticate and fetch the docs, we will need to fall back to asking users questions to fill in the gaps in the constitution. How can we keep this experience simple and not overwhelming for users?
> 4. How would a Platform engineer distribute this constitution to the teams?
> 5. How do we keep the constitution up to date as organizational standards evolve?

**Application Architecture**

```
● skill(app-modeling)

◐ Loading Platform-Engineering-Constitution.md...
  └ Constitution loaded (7 sections)

◐ Scanning feedback-service/ for infrastructure
  dependencies...

  Project structure:
    feedback-service/
    ├── requirements.txt    # fastapi, uvicorn, sqlalchemy,
    │                       # psycopg2-binary, alembic
    ├── Dockerfile          # FROM python:3.12-slim, EXPOSE 8000
    ├── docker-compose.yml  # services: app, db (postgres:16)
    └── src/
        ├── main.py         # FastAPI app with CRUD routes
        ├── models.py       # SQLAlchemy models (Feedback)
        ├── database.py     # PostgreSQL connection via
        │                   # DATABASE_URL env var
        └── schemas.py      # Pydantic request/response models

◐ Detecting infrastructure dependencies...

  Found 2 dependencies:

  ┌───────────────┬────────────────────────┬──────────────────┐
  │ Dependency    │ Source                 │ How detected     │
  ├───────────────┼────────────────────────┼──────────────────┤
  │ PostgreSQL    │ psycopg2-binary in     │ requirements.txt │
  │               │ requirements.txt,      │ + database.py    │
  │               │ DATABASE_URL in        │ connection string│
  │               │ database.py            │                  │
  │ Python App    │ Dockerfile             │ FROM python:3.12 │
  │               │ (FastAPI + Uvicorn)    │ + EXPOSE 8000    │
  └───────────────┴────────────────────────┴──────────────────┘

  ℹ No cache detected — app only uses PostgreSQL
    for persistence.

◐ Mapping dependencies to Approved IaC modules...

  ┌───────────────┬───────────────────────────────────────────┐
  │ Dependency    │ IaC modules                               │
  ├───────────────┼───────────────────────────────────────────┤
  │ PostgreSQL    │ mycompany.azurecr.io/infra/postgres:v1.2  │
  │ Python App    │ mycompany.azurecr.io/infra/python-app:v1.0│
  └───────────────┴───────────────────────────────────────────┘

? Accept these mappings? (Y/n): Y

✔ Dependencies mapped to IaC modules

◐ Verifying compliance against constitution...

  Checks:
    ✔ Cloud provider: Azure                 
    ✔ Compute: Kubernetes (AKS)             
    ✔ IaC module: from approved catalog     
    ✔ Module version: v1.2 matches catalog  
    ✖ Naming: "feedback-service"            
        Expected: mycompany-{env}-{app}-{resource}
    ✔ Tags: environment, team, cost-center 

  ℹ Verified compliance to Platform Constitution. 1 violation found.

? Auto-fix the naming violation? (Y/n): Y

  ✔ Renamed to mycompany-dev-feedback-service

✖ 1 violation found, 1 auto-fixed

◐ Generating application definition...

  ┌──────────────────────────────────────────────────────────╮
  │ Application Definition Summary                           │
  ├──────────────────────────────────────────────────────────┤
  │                                                          │
  │  📦 customer-feedback-service                            │    
  │                                                          │
  │  ┌─────────────────────────────────────────────────────┐ │
  │  │ 🗄  PostgreSQL Database                             │ │
  │  │                                                     │ │
  │  │  Name:     mycompany-dev-feedback-service-db        │ │             
  │  │  Registry: mycompany.azurecr.io/infra/postgres      │ │
  │  └─────────────────────────────────────────────────────┘ │
  │         │                                                │
  │         │ connection: DATABASE_URL (auto-injected)       │
  │         ▼                                                │
  │  ┌─────────────────────────────────────────────────────┐ │
  │  │ 🐍 Python App                                       │ │
  │  │                                                     │ │
  │  │  Name:     mycompany-dev-feedback-service-app       │ │      
  │  │  Image:    feedback-service:latest                  │ │
  │  │  Port:     8000                                     │ │
  │  └─────────────────────────────────────────────────────┘ │
  │                                                          │
  │  Naming: mycompany-{env}-{app}-{resource}                │
  │  Tags:   environment, team, cost-center                  │
  │                                                          │
  ╰──────────────────────────────────────────────────────────╯

  ℹ app.bicep generated 

? Does this look correct? (Y/n): Y

✔ Application definition generated
```

> **Behind the scenes:** The following is done in the background
> 1. The `app-modeling` skill detects dependencies. It looks for common patterns like database connection strings, ORM libraries, Docker base images, and exposed ports to infer what resources the app needs.
> 2. Matches to the Radius Resource types and approved modules in the catalog for Recipes that can provision those resources. It checks the constitution for any specific requirements around those resources and applies the practices to Recipes.
> 3. Produces [`app.bicep`](#appendix). The `connections` block wires the container to the database, so Radius automatically injects the connection string. This file acts as the deployment state that the developer doesn't need to edit it directly.

>**Open questions**:
>1. How do we handle mapping Resource type and Recipes and where do we store this mapping?
>1. How do we handle updates to the application definition? How do we ensure that changes don't break compliance?

**Application Verification**

```
● skill(app-verify)

◐ Running application verification...

  Application Audit Report
  ════════════════════════

  Summary: 0 critical · 0 warnings · 4 info

  Findings:
    ℹ [I1] PostgreSQL module from approved catalog (§5)
    ℹ [I2] Naming violation auto-fixed:
           feedback-service → mycompany-dev-feedback-service
    ℹ [I3] Explainability comments added to app.bicep
    ℹ [I4] Connection wiring: container → db (auto)

  Constitution Compliance:
    ✔ §2 Cloud Provider     — Azure
    ✔ §3 Compute Platform   — Kubernetes (AKS)
    ✔ §5 Module Catalog     — postgresql-ha v1.2
    ✔ §6 Naming Convention  — mycompany-dev-feedback-service
    ✔ §7 Required Tags      — environment, team, cost-center

  ════════════════════════
  Result: PASS — ready to deploy

? Approve and deploy? (Y/n): Y

◐ Deploying to mycompany-dev environment...

✔ Application deployed successfully

  🌐 https://mycompany-dev-feedback-service.azurewebsites.net
```

>**Open questions**:
>1. Two stages of verification should happen: 1) Application is compliant against the platform constitution and 2) Application is deployable in the target environment. 2 is dependent on Repo Radius workflow on how Radius is setup and environment is configured.

#### Edge Cases

- No constitution found: Prompt developer to create one before proceeding
- Cannot authenticate to private docs: Fall back to asking user questions to fill in the gaps in the constitution. Keep it simple.
- No dependency detected: Allow user to manually specify what resources they need and map them to the catalog
- Unapproved module detected: Flag in audit report and require explicit approval to proceed
- Violations from constitution: Auto-fix if possible or require user to provide compliant name

## Appendix

### Example: Platform Engineering Constitution

The following is an example of the `Platform-Engineering-Constitution.md` generated for MyCompany.Inc based on the E2E journey above. The constitution is the single source of truth for organizational platform standards and is consumed by Radius skills during application assembly.

```markdown

# Platform Engineering Constitution

## MyCompany.Inc

---

## 1. Organization Overview

- **Company**: MyCompany.Inc
- **Platform Team**: Cloud Infrastructure & Developer Experience
- **Last Updated**: 2026-03-05
- **Source**: https://mycompany.inc/wiki/platform-guide

---

## 2. Cloud Providers

| Provider | Status   | Regions                    |
|----------|----------|----------------------------|
| Azure    | Approved | East US, West US, West EU  |

---

## 3. Compute Platform

| Platform   | Version | Status   |
|------------|---------|----------|
| Kubernetes | 1.30    | Approved |

- **Managed Service**: Azure Kubernetes Service (AKS)
- **Node Pools**: System (Standard_D4s_v5), User (Standard_D8s_v5)

---

## 4. Infrastructure as Code

| Tool      | Version | Status   |
|-----------|---------|----------|
| Terraform | >= 1.6  | Approved |

- **State Backend**: Azure Storage Account
- **Module Registry**: mycompany.azurecr.io

---

## 5. Approved Module Catalog

All infrastructure must be provisioned using approved modules from the
organization's private registry. Modules not in this catalog require
platform team approval before use.

| Module               | Version | Registry Path                              |
|----------------------|---------|--------------------------------------------|
| postgresql-ha        | v1.2    | mycompany.azurecr.io/infra/postgres        |
| redis-cluster        | v2.0    | mycompany.azurecr.io/infra/redis           |
| mysql-ha             | v1.1    | mycompany.azurecr.io/infra/mysql           |
| mongodb-replica      | v1.0    | mycompany.azurecr.io/infra/mongodb         |
| rabbitmq-cluster     | v1.3    | mycompany.azurecr.io/infra/rabbitmq        |
| kafka-cluster        | v2.1    | mycompany.azurecr.io/infra/kafka           |
| python-app           | v1.0    | mycompany.azurecr.io/infra/python-app      |
| storage-account      | v1.4    | mycompany.azurecr.io/infra/storage         |
| key-vault            | v1.2    | mycompany.azurecr.io/infra/keyvault        |
| service-bus          | v1.0    | mycompany.azurecr.io/infra/servicebus      |
| container-registry   | v1.1    | mycompany.azurecr.io/infra/acr             |
| dns-zone             | v1.0    | mycompany.azurecr.io/infra/dns             |

---

## 6. Naming Convention

**Pattern**: `{company}-{env}-{app}-{resource}`

| Segment      | Description                          | Example            |
|--------------|--------------------------------------|--------------------|
| `{company}`  | Organization prefix                  | mycompany          |
| `{env}`      | Deployment environment               | dev, staging, prod |
| `{app}`      | Application name (kebab-case)        | feedback-service   |
| `{resource}` | Resource type suffix                 | db, app, cache     |

**Example**: `mycompany-dev-feedback-service-db`

---

## 7. Required Tags

All cloud resources must include the following tags:

| Tag            | Description                        | Example              |
|----------------|------------------------------------|----------------------|
| `environment`  | Deployment environment             | dev, staging, prod   |
| `team`         | Owning team                        | platform, backend    |
| `cost-center`  | Billing cost center                | CC-1234              |
```

### Example: Generated `app.bicep`

The following is an example of the `app.bicep` generated by the `app-modeling` skill for Maria's customer feedback service.

```bicep
import radius as radius

@description('The Radius environment ID for deployment')
param environment string

@description('The Radius application ID')
param application string

resource db 'Radius.Datas/postgreSqlDatabases@2025-08-01-preview' = {
  name: 'mycompany-dev-feedback-service-db'
  properties: {
    environment: environment
    application: application
  }
}

resource app 'Radius.Compute/containers@2025-08-01-preview' = {
  name: 'mycompany-dev-feedback-service-app'
  properties: {
    environment: environment
    application: application
    container: {
      image: 'feedback-service:latest'
      ports: {
        http: {
          containerPort: 8000
          protocol: 'TCP'
        }
      }
    }
    connections: {
      database: {
        source: db.id
      }
    }
  }
}
```
