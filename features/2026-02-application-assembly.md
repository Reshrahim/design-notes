# Topic: Application Assembly

* **Author**: Reshma Abdul Rahim (@reshrahim)

## Topic Summary

Radius today requires significant upfront setup before developers can deploy their applications. Platform engineers must manually define Resource Types, register Recipes, and configure Environments. Developers must then learn the Radius resource model and re-author their application definitions in Bicep or YAML before they can realize any value from the platform. This setup cost creates a steep adoption barrier that discourages both new users and organizations with large existing application portfolios.

Application Assembly is a new layer in Radius that introduces an application-intent layer, translating existing codebases into platform-compliant deployments. Application Assembly automatically discovers application components, infers the platform requirements and standards, and generates a deployable Radius application definition — turning what was a multi-day onboarding process into a single command.

### Principles

1. **Developer-first**: Optimize for a seamless developer experience, while honoring platform engineering standards. The default path should be simple for application teams and have basic compliance with organizational policies.

2. **Cognitive load reduction**: Use developer language and hide internal concepts (Resource Types, Recipes, Bicep) by default. Users should get value without learning the Radius resource model.

3. **Progressive adoption**: Generate a working starting point that is easy to extend and customize as users learn more. The initial experience is low-friction, with a clear path to deeper control.

4. **Trust and transparency**: Always show what was inferred, from where, and with what confidence. Require review before deploy, and make it easy to accept, edit, or reject each inference.

5. **Time to value**: Move from “existing app” to “running on Radius” in minutes, but never at the expense of correctness or safety.

### Goals

1. Introduce an application-intent layer that allows developers to deploy existing and new applications to Radius without needing to understand internal platform constructs such as Resource Types or Recipes.
2. Reduce onboarding friction for users and accelerate time to value by enabling an experience that automatically discovers application components, infers platform requirements, and generates deployable Radius application definitions.
3. Provide a deterministic and transparent assembly experience that surfaces inferred topology, authentication models, and platform mappings for review before deployment.

### User Journey

#### Scenario 1: As an enterprise application developer, I want to assemble my app from source code and configuration, so that I can deploy it on Radius with confidence that it meets platform standards.

**Prerequisites**: Radius is initialized in the repository via `rad init`.

**Assumptions**: The platform engineering team has defined a Platform Constitution that includes approved Terraform modules, naming conventions, and authentication preferences else we fall back to safe defaults.

The assembly workflow is split into two commands with a clear boundary: **discover** produces a human-editable YAML manifest, **model** consumes it and generates a deployable Bicep `app.bicep`.

```
rad app discover  →  rad app model  →  radius/app.bicep
```

### Discover and understand application using `rad app discover`

Scans the repository, gathers inputs from the developer, and writes a `radius/app.yaml` manifest that captures the full application topology, secrets, and auth model. All generated artifacts are placed in the `radius/` folder for further edits.

**Step 1 — Scan and detect**

Radius scans the repository to detect services and their dependencies in a single pass.

```
$ rad app discover

⠋ Scanning repository to detect:
  ✔ Services and Runtimes
  ✔ Dependencies(databases, caches, external APIs)
  ✔ Authentication patterns
  ✔ Platform artifacts (IaC modules, docs, deploy configs)

All detected components and inferences will be presented for review before proceeding.
```

Once done, Radius renders an architecture diagram and evidence tables:

```
 Application Architecture — my-app (inferred)
 ─────────────────────────────────────────────

          ┌──────────┐        HTTP :8080        ┌──────────┐
  :3000   │          │─────────────────────────▶│          │
 ────────▶│   web    │                          │   api    │
  public  │ (Node.js)│                          │  (Go)    │
          └──────────┘                          └──┬─┬─┬──┘
                                                   │ │ │
                                        ┌──────────┘ │ └──────────┐
                                        ▼            ▼            ▼
                                 ┌────────────┐ ┌────────────┐ ┌──────────────┐
                                 │ PostgreSQL │ │   Redis    │ │ Azure OpenAI │
                                 │ (database) │ │  (cache)   │ │    (ai)      │
                                 └────────────┘ └────────────┘ └──────────────┘
```

```
 Detected components
 ────────────────────
 # │ Component    │ Type       │ Runtime │ Port │ Connects to              │ Authentication      │ Source
 ──┼──────────────┼────────────┼─────────┼──────┼──────────────────────────┼─────────────────────┼──────────────────────────────
 1 │ web          │ service    │ Node.js │ 3000 │ api                      │ —                   │ ./web/Dockerfile:14
 2 │ api          │ service    │ Go      │ 8080 │ postgresql, redis,       │ —                   │ ./api/Dockerfile:22
   │              │            │         │      │ azure-openai             │                     │
 3 │ postgresql   │ dependency │ —       │ —    │ —                        │ connection-string   │ ./api/.env:1 (DATABASE_URL)
 4 │ redis        │ dependency │ —       │ —    │ —                        │ connection-string   │ ./api/.env:2 (REDIS_HOST)
 5 │ azure-openai │ dependency │ —       │ —    │ —                        │ managed-identity    │ ./api/internal/ai/client.go:14

 Actions: [✔ accept all / ✎ edit / ✘ remove / + add]

  Confirm detected topology? [Y/n]: Y
  ✔ Application topology confirmed.
 
```

**Step 2 — Platform constitution**

Radius detects the Platform Constitution — a markdown file maintained by the platform team that defines approved IaC modules, naming conventions, labels, and other organizational standards. See [Appendix: Platform Constitution (draft)](#appendix-platform-constitution-draft) for a full example.

```
 Platform constitution
 ─────────────────────

 ⠋ Scanning for platform constitution...
   ✔ Found ./platform-constitution.md

 ⠋ Reading ./platform-constitution.md...

 Detected IaC modules:
 ──┬──────────────────────────────────────────────┬─────────┬────────────────────────────────────
 # │ Module                                       │ Version │ Provisions
 ──┼──────────────────────────────────────────────┼─────────┼────────────────────────────────────
 1 │ myorganization/aks-container/azurerm         │ v1.0.0  │ azurerm_kubernetes_cluster_node_pool
 2 │ myorganization/postgresql-flexible/azurerm   │ v2.1.0  │ azurerm_postgresql_flexible_server
 3 │ myorganization/redis-cache/azurerm           │ v1.3.0  │ azurerm_redis_cache
 4 │ myorganization/vnet/azurerm                  │ v3.0.1  │ azurerm_virtual_network
 5 │ myorganization/keyvault/azurerm              │ v1.5.0  │ azurerm_key_vault
 6 │ myorganization/identity/azurerm              │ v1.0.2  │ azurerm_user_assigned_identity
 ──┴──────────────────────────────────────────────┴─────────┴────────────────────────────────────

 Standards applied:
   ✔ Registry: app.terraform.io/myorganization
   ✔ Naming:   {env}-{app}-{component}
   ✔ Labels:   app.kubernetes.io/part-of, app.kubernetes.io/managed-by

 ✔ Platform constitution loaded.
```

**Step 3 — Write `radius/app.yaml`**

Radius writes the discovery results to `radius/app.yaml`. The developer can review and edit this file before running `rad app model`.

```
 ✔ radius/app.yaml written.
```

The generated `radius/app.yaml`:

```yaml
# radius/app.yaml — generated by rad app discover
# Edit this file, then run: rad app model

application:
  name: my-app
  registry: ghcr.io/myorganization
  labels:
    app.kubernetes.io/part-of: my-app
    app.kubernetes.io/managed-by: radius
  naming: "{env}-{app}-{component}"

services:
  - name: web
    runtime: nodejs
    dockerfile: ./web/Dockerfile
    port: 3000
    public: true
    dependencies:
      - target: api
        protocol: http
        port: 8080

  - name: api
    runtime: go
    dockerfile: ./api/Dockerfile
    port: 8080
    dependencies:
      - target: postgresql
        auth: connection-string
      - target: redis
        auth: connection-string
      - target: azure-openai
        auth: managed-identity

platform:
  constitution: ./platform-constitution.md
```

> The developer can edit `radius/app.yaml` at any point — add services, change auth models, rename secrets, adjust dependencies — and then proceed to modeling.

---

### Model the application via  `rad app model`

Reads `radius/app.yaml`, matches each component to a Radius resource type and recipe, and generates deployable artifacts into the `radius/` folder.

```
$ rad app model

 Reading radius/app.yaml...
 ✔ 2 services, 3 dependencies, 4 secrets loaded.
 ✔ Detected provider: Azure (from Terraform modules)

 Application Architecture — my-app (modeled)
 ─────────────────────────────────────────────

          ┌──────────────────────┐  HTTP :8080  ┌──────────────────────┐
  :3000   │ web                  │─────────────▶│ api                  │
 ────────▶│ Radius.Core/         │              │ Radius.Core/         │
  public  │   containers         │              │   containers         │
          │ aks-container v1.0.0 │              │ aks-container v1.0.0 │
          └──────────────────────┘              └──┬───┬───┬───────────┘
                                                   │   │   │
                                        ┌──────────┘   │   └──────────┐
                                        ▼              ▼              ▼
                              ┌──────────────────┐ ┌──────────────────┐ ┌───────────────────────┐
                              │ postgresql       │ │ redis            │ │ azure-openai          │
                              │ Radius.Data/     │ │ Radius.Data/     │ │ Radius.AI/            │
                              │   sqlDatabases   │ │   redisCaches    │ │   openAIModels        │
                              │ postgresql-      │ │ redis-cache      │ │ cognitive-services    │
                              │   flexible v2.1.0│ │   v1.3.0         │ │   /account v0.9.0     │
                              └──────────────────┘ └──────────────────┘ └───────────────────────┘

 
 Matched resource types and recipes
 ───────────────────────────────────

 # │ Component    │ Resource type              │ Recipe Spurce
 ──┼──────────────┼────────────────────────────┼───────────────────────────────────────────────────────────────────────
 1 │ web          │ Radius.Core/containers     │ app.terraform.io/myorganization/aks-container/azurerm  v1.0.0
 2 │ api          │ Radius.Core/containers     │ app.terraform.io/myorganization/aks-container/azurerm  v1.0.0
 3 │ postgresql   │ Radius.Data/sqlDatabases   │ app.terraform.io/myorganization/postgresql-flexible/azurerm  v2.1.0
 4 │ redis        │ Radius.Data/redisCaches    │ resource-type-contrib/avm/res/cache/redis  v0.8.0
 5 │ azure-openai │ Radius.AI/openAIModels     │ resource-type-contrib/avm/res/cognitive-services/account  v0.9.0

 [✔ accept all / line # to edit]

 Accept all? [Y/n]: Y
 ✔ All resource types and recipes source confirmed.

  Generating...
 ✔ radius/app.bicep written.
 ✔ radius/types.yaml written.
 ✔ radius/recipe-pack.yaml written.

 Summary
 ───────
 App: my-app   Services: 2   Dependencies: 3   Secrets: 4
 Resource types: 4 (radius/types.yaml)
 Recipe pack: 4 modules (radius/recipe-pack.yaml)

 Next steps:
   rad app plan           Review the generated plan
   rad deploy app.bicep   Deploy when ready
```

### Learnings and Areas under Exploration

#### Deterministic Resource Types matching

The current implementation performs deterministic matching between detected dependencies and resource types from the `resource-types-contrib` repository. We need to enrich the `resource-types-contrib` repository to expand coverage of common services.

Areas under exploration:

1. Deriving resource types from authoritative infrastructure libraries (AVM, Terraform Registry).

2. Introducing a human-reviewed promotion pipeline.

3. Enabling community contributions through templated onboarding flows.

#### Multi-source Recipe generation

Recipes are generated from multiple sources with clear precedence rules. Enterprise Internal repositories and registries are ranked higher in user preference than public registries, suggesting organizations want recipes that reflect their security posture and cost policies.

#### Source Code Parsing

The current prototype relies on deterministic static parsing (for example, analyzing go.mod in Go) to identify dependencies and imports. This approach provides predictable results and low latency but may miss implicit dependencies or how components are actually used within the application. Maintaining language-specific parsers across ecosystems also introduces long-term operational overhead.

To address these gaps, a hybrid strategy is being explored: static parsing remains the primary and deterministic first pass, while LLM-assisted analysis is selectively applied to cases where static rules lack context (for example, inferring how a dependency like etcd is used). LLM usage introduces additional cost and latency, so it will be applied sparingly. For the initial prototype, a deterministic parsing approach is sufficient and preferred.

#### Skills based Architecture that enables multiple Interfaces

We believe that all the automatic discovery and generation capabilities will be composable skills invoked via a shared engine so that a developer can use CLI or an AI coding agent to deploy the application using Radius. This is still under experimentation.

---

## Appendix: Platform Constitution (draft)

The Platform Constitution is a markdown file maintained by the platform engineering team. Radius parses this file during `rad app discover` to apply organizational standards automatically.

Below is an example `platform-constitution.md`:

````markdown
# Platform Constitution — myorganization

## Terraform Registry

All approved Terraform modules are published to the organization's private registry.

- **Registry URL**: `app.terraform.io/myorganization`
- **Authentication**: Terraform Cloud token (`TF_TOKEN`) must be stored in Azure Key Vault and accessed via managed identity; plain-text tokens are not permitted.

## Naming Convention

All resources must follow this naming pattern:

```
{env}-{app}-{component}
```

Example: `dev-my-app-postgresql`

## Labels

All Kubernetes resources must include the following labels:

| Label | Value | Required |
|-------|-------|----------|
| `app.kubernetes.io/part-of` | Application name | Yes |
| `app.kubernetes.io/managed-by` | `radius` | Yes |

## Authentication Preferences

Preferred authentication methods for infrastructure dependencies, in order of preference:

1. **Managed identity** — use for all Azure services that support it
2. **Connection string** — use when managed identity is not supported
3. **API key** — use only as a last resort

## Security Policies

- All secrets must be stored in Azure Key Vault; plain-text environment variables are not permitted in production.
- All inter-service communication must use mTLS.
- Public endpoints require WAF and rate limiting.
````
