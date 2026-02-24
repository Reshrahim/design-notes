# Topic: Application Assembly

* **Author**: Reshma Abdul Rahim (@reshrahim)

## Topic Summary

Radius today requires significant upfront setup before developers can deploy their applications. Platform engineers must manually define Resource Types, register Recipes, and configure Environments. Developers must then learn the Radius resource model and re-author their application definitions in Bicep before they can realize any value from the platform. This setup cost creates a steep adoption barrier that discourages both new users and organizations with large existing application portfolios.

Application Assembly is a new layer in Radius that introduces an application-intent layer, translating existing codebases into platform-compliant deployments. Application Assembly automatically discovers application components, infers the platform requirements and standards, and generates a deployable Radius application definition.

### Principles

1. **Developer-first**: Optimize for a seamless developer experience, while honoring platform engineering standards. The default path should be simple for application teams and have basic compliance with organizational policies.

2. **Cognitive load reduction**: Use developer language and hide internal concepts (Resource Types, Recipes, Bicep) by default. Users should get value without learning the Radius resource model.

3. **Progressive adoption**: Generate a working starting point that is easy to extend and customize as users learn more. The initial experience is low-friction, with a clear path to deeper control.

4. **Trust and transparency**: Always show what was inferred, from where, and with what confidence. Require review before deploy, and make it easy to accept, edit, or reject each inference.

### Goals

1. Introduce an application-intent layer that allows developers to deploy existing and new applications to Radius without needing to understand internal platform constructs such as Resource Types or Recipes.
2. Reduce onboarding friction for users and accelerate time to value by enabling an experience that automatically discovers application components, infers platform requirements, and generates deployable Radius application definitions.
3. Provide a deterministic and transparent assembly experience that surfaces inferred topology, authentication models, and platform mappings for review before deployment.

### User Journey

### Scenario 1 : As an enterprise application developer, I want to create a new application using copilot that match the enterprise platform standards

Context: A developer is creating a new application using Copilot CLI. They want to ensure that the application is compliant with the enterprise platform standards defined by the platform engineering team, such as using approved Terraform modules, following naming conventions, and adhering to authentication preferences.

### Learnings and Areas under Exploration

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
