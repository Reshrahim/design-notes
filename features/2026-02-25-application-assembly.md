# Topic: Application Assembly

* **Author**: Reshma Abdul Rahim (@reshrahim)

## Topic Summary

Enterprise developers today face a fundamental disconnect between writing application code and deploying it in compliance with organizational platform standards. Platform engineers spend significant time defining infrastructure policies, approved modules, and deployment conventions, but developers have no automated way to discover or apply these standards. The result is a slow, manual, and error-prone process where compliance is enforced reactively instead of being built in from the start.

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



## User profile and challenges

Application Assembly targets two primary personas who collaborate across the platform engineering workflow: **enterprise application developers** who build and ship applications, and **platform engineers** who define and enforce organizational infrastructure standards.

### User persona(s)

**Enterprise Application Developer**

Application developers in enterprises who are building new applications and want to deploy them in compliance with organizational standards.

- **Established developers**: Experienced engineers bootstrapping new services who want to move fast without manually researching which infrastructure modules are approved, how to wire up dependencies and how to handle secrets. They want to focus on their application code and let the platform handle the rest.

- **New developers**: Engineers from adjacent disciplines (data analysts, front-end developers, ML engineers) or new hires who need to stand up compliant applications but lack familiarity with the organization's infrastructure tooling and policies.

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

### User Journey

#### End-to-End Flow Summary

```
Platform Engineer                    Developer                    Platform 
      │                                  │                              │
      ▼                                  │                              │
 ┌──────────────┐                        │                              │
 │  platform-   │                        │                              │
 │ constitution │                        │                              │
 │    skill     │                        │                              │
 └──────┬───────┘                        │                              │
        │                                │                              │
        ▼                                │                              │
 Constitution.md                         │                              │
        │                                │                              │
        └────────────────────────────────▶                              │
                                         │                              │
                                    ┌────┴─────┐                        │
                                    │  app-    │                        │
                                    │ modeling │                        │
                                    │  skill   │                        │
                                    └────┬─────┘                        │
                                         │                              │
                                         ▼                              │
                                    app.bicep                           │
                                         │                              │
                                         └──────────────────────────────▶
                                                                        │
                                                                   ┌────┴──────┐
                                                                   │   app-    │
                                                                   │ verify   │
                                                                   │  skill   │
                                                                   └────┬──────┘
                                                                        │
                                                                        ▼
                                                                   Audit Report
```

#### Prerequisites

- Copilot CLI installed
- Skills added to `.github/skills/` or configured in Copilot

#### Phase 1: Platform Constitution (Scenario 2)

> **Persona:** Platform Engineer or Developer  
> **Goal:** Define organizational standards in one place that can be automatically applied during application assembly

A platform engineer typically drives this step, but a developer can also trigger it for example, when no Platform Constitution exists yet and the developer wants to bootstrap one to get started.

##### Prompt

**Developer prompt (no constitution exists yet):**

```
Build me a simple Node.js todolist app
```

When no Platform Constitution is found, the system prompts the developer to generate one before proceeding with application assembly.

**Platform engineer prompt (direct):**

```
Generate a Platform Engineering Constitution for my organization
```

##### Expected Behavior

Copilot asks questions one at a time, organized by category:

**Organization**
1. "What is your organization name?" → `MyCompany Inc.`

**Cloud & Compute**
2. "What cloud providers does your organization use?" → `Azure`
3. "What compute platforms do you target?" → `Kubernetes (AKS)`
4. "What cloud regions are approved?" → `East US, West US 2`

**Infrastructure as Code**
5. "What IaC tooling does your platform team use?" → `Bicep`
6. "List approved IaC modules with registry path" → `mycompany.azurecr.io`
7. "Do you have existing infrastructure policies?" → `No`

**Compliance & Governance**
8. "Any compliance requirements?" → `SOC 2`
9. "Naming conventions?" → `{org}-{env}-{service}-{resource}`
10. "Required tags?" → `environment, team, cost-center`

**Networking & Security**
11. "Network architecture constraints?" → `Hub-spoke with private endpoints`
12. "How are secrets managed?" → `Azure Key Vault`

##### Expected Output

`Platform-Engineering-Constitution.md` created at repo root with sections:
1. Organization Overview
2. Cloud Providers (Azure)
3. Compute Platform (AKS)
4. Infrastructure Policies
5. Infrastructure as Code (Bicep/Terraform, module catalog)
6. Deployment Standards
7. Network Architecture
8. Security & Secrets
9. Appendix

#### Phase 2: Application Modeling (Scenario 1)

> **Persona:** Developer  
> **Goal:** Create a new application that is compliant with platform standards

##### Prompt

```
Build me a todo app and deploy it
```

The developer describes what the app should do, not how it should be built. The system scaffolds the application code, infers infrastructure dependencies (e.g., a database for persistence, a cache for performance), and automatically invokes the application-modeling skill to generate a compliant, deployable application.

Copilot generates a Node.js app with inferred dependencies:

```
sample-app/
├── package.json        # has "pg" and "ioredis" dependencies
├── Dockerfile          # FROM node:20-alpine, EXPOSE 3000
├── docker-compose.yml  # services: app, db (postgres:15), cache (redis:7)
└── src/
    └── index.js        # connects to postgres and redis
```

##### Expected Behavior

1. **Reads constitution** — finds `Platform-Engineering-Constitution.md`
2. **Scans files** — detects `package.json`, `Dockerfile`, `docker-compose.yml`
3. **Detects dependencies:**
   - PostgreSQL (`pg` in package.json)
   - Redis (`ioredis` in package.json)
   - Node.js container (`Dockerfile` + docker-compose `app` service)
4. **Maps to Radius types** (from resource-types-contrib):
   - `Radius.Data/postgreSqlDatabases`
   - `Radius.Data/redisCaches`
   - `Radius.Compute/containers`
5. **Selects Recipes from module catalog:**
   - PostgreSQL → `postgresql-ha` (approved, `mycompany.azurecr.io/infra/postgresql:v1.2.0`)
   - Redis → `redis-cluster` (approved, `mycompany.azurecr.io/infra/redis:v2.0.0`)
6. **Complies with standards** — ensures cloud provider, compute platform, naming conventions, and tags match constitution
7. **Verifies via MCP** — checks if recipes are registered in environment
8. **Generates Bicep** — `app.bicep` with explainability comments

#### Phase 3: Application Verification (Scenario 2)

> **Persona:** Developer 
> **Goal:** Verify the generated application meets organization standards

##### Prompt

```
Verify and audit this application meets our platform standards
```

The system automatically invokes the application-verification skill based on the user's intent.

##### Expected Behavior

1. **Reads constitution** — loads standards
2. **Reads generated Bicep** — parses `app.bicep`
3. **Extracts approved module catalog** from constitution
4. **Checks each resource** against constitution:
   - Cloud provider alignment
   - Approved IaC modules
   - Naming conventions
   - Tags/labels
5. **Verifies via MCP** — recipe registration, resource type registration
6. **Generates audit report**

##### Expected Output

```
> Verify and audit this application meets our platform standards

✔ Loading constitution... done
✔ Parsing app.bicep... done
✖ Checking resources against standards... 1 critical finding

Application Audit Report
========================

Summary:  1 critical · 0 warnings · 2 info

Findings:

  ✖ [C1] Unapproved IaC used for PostgreSQL
      File: resource-types/contrib/postgreSQL.bicep
      Expected: mycompany.azurecr.io (Constitution Section 5)

  ℹ [I1] All other recipes from approved module catalog
  ℹ [I2] Explainability check passed

? Do you want to proceed with deployment? (Y/N)
```

#### Edge Cases

- No constitution found: Prompt user to create one before proceeding with assembly.
- No dependencies detected: 
- Unapproved module detected: Flag in audit report and require explicit approval to proceed.

## Learnings and Areas under Exploration

#### 1. Scenario Clarification: Greenfield vs Brownfield

Our focus for the initial Application Assembly experience is greenfield application development, where enterprise developers are building new applications from scratch using AI coding assistants. We cater to two primary groups of developers creating new applications:

- **Established developers**: Enabling experienced developers to bootstrap new applications that automatically adhere to organizational platform standards and practices.
- **New developers**: Enabling other disciplines (analysts, designers, sales) unfamiliar with infrastructure to scaffold compliant applications from day one.

Brownfield scenarios are still important but present challenges around discovery of the application and infrastructure needs spread across the codebase, wikis, institutional knowledge, and partially compliant deployed infrastructure. We will explore brownfield assembly in future iterations after validating the core assembly experience in greenfield contexts.

#### 2. Deterministic Application Modeling

We want to ensure there is maximum determinism in the assembly process so that developers can trust the results and understand how the application is modelled.

*Deterministic components*:
- Resource type matching from `resource-types-contrib` repository
- Recipe selection based on approved modules from platform constitution or from `resource-types-contrib` repository
- CLI or API calls to use Radius capabilities (`rad resource-type show` or `rad resource-type list`)

That being said, there are certain aspects of the assembly process that may require LLM inference beyond deterministic rules, especially when it comes to understanding the dependencies and matching them to the correct resource types and recipes. For example, if a dependency is detected but does not have a clear mapping to a resource type or a recipe, we could use an LLM to suggest potential matches based on the dependency's characteristics and usage patterns.

#### 3. Skills Installation and Distribution

We are building the assembly capabilities as composable skills that can be invoked via AI coding agent or CLI. We need to determine the best way to distribute these skills to users and ensure they are easily discoverable and updatable. Copilot supports project skills stored within `.github/skills` folder and personal skills stored in agent folders `.copilot/skills`.

In Copilot CLI, a user could add Radius skills using the below command:

```
/skills add radius-project/radius-skills
```

Other options for distribution include:
| Method | Description|
|--------|------|------
| `skills.sh` script | A simple scripts that copies the necessary skills from the centralized Radius repository to the user's `.github/skills` folder.
| `rad cli` | Integrated experience within the Radius CLI |
| GitHub's recommended way | This could be an option that GitHub provides in the future for sharing skills across users and organizations. |

#### 4. `app.bicep` as State file (Hidden from User)

We want to position the generated `app.bicep` as an implementation detail and source of truth that users don't need to interact with directly. The generated `app.bicep` can function similarly to Terraform state file which can be iterated by Radius before finalizing it for deployment.

This state file will be modified at various stages of the assembly and deployment process, for example:

- During application modeling, `app.bicep` is generated based on the inferred application topology and platform mapping
- During user review, any changes made by the user to the inferred topology or platform mapping will be reflected in the `app.bicep`
- During platform compliance checks, any adjustments needed to meet organizational policies can be made in the `app.bicep` before final deployment.
