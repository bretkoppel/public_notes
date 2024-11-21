# Chapter 1: Why Platform Engineering is Becoming Essential

- Common criticisms of central platform teams: Offerings are too hard to use; not customer-focused, instead focusing on their own priorities; lack of stability.
- Common criticisms of app team access: Lower shared knowledge means lower efficienty; force more SRE, DevOps specialists.
- Defns
  - Platform: A foundation of self-service APIs, tools, services, knowledge, support that are arranged as a compelling internal product. Autonomous app teams use the platform to deliver product features faster, with reduced coordination. What isn't a platform? Documentation alone; "The could", whose product volume is too overwhelming to be seen as a coherent platform.
  - Platform Engineering: Discipline of developing and operating platforms with a goal to manage overall system complexity in order to deliver leverage to the business. Meets goal by taking a curated product approach to developing platforms as software-based abstractions that serve a broad base of app devs.
  - Leverage: The work of a few engineers on a platform team reduces the work of the greater org. Core value of PlatEng team. Achieved via making application engineers deliver business value more productively; making eng org more efficient by eliminating duplicate work across app eng teams.
  - Product: Essential to view platform as a product, meaning that they must be compelling, customer-centric offerings. Customers will ask for a broad range of features, but the product is deliberately curated through what it does and what it leaves out.

## The Over-General Swamp

- Many types of internal platforms. Book applies to all, but is most focused on infrastructure and developer tooling spaces. These spaces are most closely integrated with public cloud, OSS, whose proliferation have increased cost of ownership of systems over time - applications are easier to build but harder to maintain.
- Post-initial-dev maintenance is estimated to account for 60-75% of the lifetime cost of software - Required upgrades / security patches; retesting; new version migrations for deps; etc. Cloud+OSS amplify these costs by providing many new _primitives_(general-purpose building blocks that are loosely integrated) which need "glue"(integration code, one-off automation, configuration, and management tools).
- The _over-general swamp_ forms as the "glue" spreads - app teams make independent choices across all of the primitives, selecting the ones that allow them to meet goals most quickly, then create custom glue to bolt those primitives together. The resulting swamp makes evolving applications more difficult as each piece requires different maintenance work / cadence. Integration and testing burden grows. Platforms abstract a limited set of OSS / Vendor choices in an opinionated manner, making for less smeared glue.

## How We Got Stuck in the Over-General Swamp

### Industry Change #1: Explosion of Choice

- Internet age drove increased demand for software, thus hardware. Initially met by dedicated hardware, thus infrastructure engineering - more servers and networking gear, etc. During this period, app devs were frustrated by constantly changing menu of server choices; data center capacity issues; hardware-related operationl issues that they couldn't get help with("Nothing in system logs, check your software").
- Public cloud brings api-driven arch choices separate from an unresponsive infrastructure engineering org / team. Orgs recognize arch complexity, security, reliability, cost risks but adopt anyway.
- Result has leaned more toward IaaS than PaaS. PaaS struggles to support a wide range of applications and integrate with existing apps / infra, so companies building software in-house at scale embrace IaaS for flexibility.
  - IaaS: Vendor apis are used to provision a virtualized environment with various other infrastructure primitives, which run an application much as they would run on physical hosts.
  - PaaS: Vendor takes full ownership of operating the app's infrastructure, offering higher-level abstractions instead of primitives. The app runs in a scalable sandbox.
- Kubernetes illustrates how PaaS and IaaS have failed to meet enterprise needs: Attempts to simplify IaaS ecosystem by forcing apps to be "cloud native", thus needing less infrastructure-specific glue. The breadth of its scope has yielded a leaky abstraction, trading YAML and Terraform glue for detailed configuration glue. Kubernetes is also an example of OSS complexity: Instead of paying a vendor for dev tools and middleware, app devs face a huge number of options, many of which are good for their use case and quick launch but not applicable to other org use cases, leading to ongoing maintenance costs of these niche choices.

### Industry Change #2: Higher Operational Needs

- Pre-internet, two roles
  - Software Developers responsible for arch, coding, testing. Often resulted in throw-it-over-the-wall monolithic distributions.
  - System Administrators responsible for production operation of software.
- Internet increased need for 24/7 operational support, thus _operations engineering_ teams, often staffed by early career sys admins. Agile reduces the role of these teams as a faster release cycle surfaced additional tension between operating teams and developing teams. Outages became more contentious, while also making it harder to point fingers due to blurring the lines of responsibility.
- DevOps intended to integrate app dev + operations, requiring both culture change and tech change. However, operational work remained, and the work was approached in two ways:
  - Split: Ops and Devs remain split, but ops does some amount of dev, particularly around creating glue for infra integration and deployment. Thus, the old operations team was now a DevOps team.
  - Merged: Ops and Devs merge and share in the operational work, including on-call rotations. Could be successful with 100% App Devs but also with more x-functional teams that included specialists to own infra and deployment glue(i.e., DevOps engineers).
- In parallel to DevOps, Google moves to SRE to help guide orgs through the operational complexity of making DevOps work. However, SRE has had limited success outside of Google: Heavyweight processes; Reliance on Google-specific culture and org focus. Thus, the challenge remains: How to ensure 24/7 operational support in a sustainable way that limits the need for dedicated DevOps/SRE engineers/teams, enabling app dev teams to deploy and operate it themselves.
  - See [6 Reasons You Don’t Need an SRE Team](https://log.andvari.net/6reasons.html) ([Archive link](https://archive.ph/gzOVp)): "The next stage in removing our production training wheels as an industry is to tear down the fence between SRE and Product Engineering, and make rational investments in reliability as a mindset, based on specific needs."

### Result: Drowning in the Swamp

- More app teams driven by the need to deliver quickly making potentially short-sighted choices over a larger landscape of tools.
- Rise of cybersecurity needs has driven additional pressure to evolve / update infrastructure / OSS, often driving additional change needs at the "glue" layer for app devs.
- Eventually, maintenance needs and greenfield project choices for new tech create a hard-to-navigate productivity swamp.

## How Platform Engineering Clears the Swamp

- Having hired more people in infra, tools, DevOps, SRE spaces but still unable to keep up with new complexity. App devs become less productive. Platform to manage complexity sounds great... But building platforms takes significant investment: Costs to build and support them, overhead associated with limiting app dev teams' choices. Additional potential org cost if you need to re-org, change roles, and roll out the new focus.

### Limiting Primitives While Minitmizing Overhead

- Explosion of choice _has_ allowed devs to ship more quickly with more autonomy and ownership over their systems. These benefits are often forgotten as companies look to reduce support burden and long-term costs from diversity of choices. Instead, leadership appeals to authority: DBREs are experts in databases, so they have a natural right to choose app databases; Architects decide on software tools and packages; CTO decides everything. These experts will tend to struggle to understand biz needs well enough to make optimal choices over time, so app teams will suffer.
- Platform engineering recognizes that app teams should have systems they enjoy using provided by teams that are responsive to them as customers, not just focused on cost reduction or their own support burden. Instead of authority-driven standards, platform eng takes a customer-focused product approach that curates a small set of primitives that meet a broad range of reqs. Requires compromises in light of biz realities, incremental delivery of good platform arch, willingness to partner directly with app teams and listen to what they need. When done well, should yield demonstrated leverage of using platform-provided offerings rather than authority-driven choices.

### Reducing Per-Application Glue

- In addition to reducing number of primitives, platform engineering aims to reduce the coupling glue of the remaining primitives by removing most of the application-level glue in favor of abstracted platform capabilities that meet broader needs.
  - OSS + Cloud offerings often expose their complexity via configuration params which, if not chosen correctly, will eventually drive production issues. As app teams called for increased flexibility in IaaS, many companies decided to give those teams power and responsibility to provision, thus driving a need to understand that IaaS more intimately and to learn to provision in a repeatable manner(e.g., by using _Terraform_). In practice, this has often lead to the following progression:
    - Engineers don't want to learn a whole new toolset for infrequent tasks, so the work would be offloaded to new hires or engineers specifically interested in DevOps. Over time, as those people churned, the work was less understood and more poorly implemented.
    - As org recognizes this pattern, leadership centralizes the work across multiple teams or the whole org... but not by building a platform, instead simply by pulling all Terraform engineers into a team that writes Terraform for others.
    - As centralized Terraform teams coalesce, they are still driven by the feature needs of app teams, making them mostly ticket-pushers. Devs who _could_ better architect Terraform abstractions don't stick around as they're constrained by feature needs. Terraform layer devolves, slowing down app teams and creating security maintenance issues.
- Instead of offering a low-coherence centralized Terraform writing team, need to focus on evolving these experts into a platform team who understands customer needs; develops opinions on which solutions to offer rather than provisioning anything requested; thinks about what they can build to go beyond simply the provisioning step. Instead of app teams hiring their own DevOps and SRE engineers to support infra, a platform team can pool the experts and give them the responsibility of identifying broader solutions for the company. Thus, they can support one-off changes while also abstracting the underlying complexity.

### Centralizing the Cost of Migrations

- Applications and primitives have long, independent lifetimes, during which each will undergo many changes. Platform engineering reduces associated costs by:
  - Reducing the diversity of OSS and cloud systems in use: Fewer primitives => less need for migrations.
  - Encapsulating OSS and vendor systems with APIs: While platform APIs are often imperfect at encapsulating all aspects of the underlying, their APIs are often good enough to protect appilcations from needing to change with the underlying systems.
  - Creating observability of platform usage: Platforms can standardize collection of metadata about their own use and the use of underlying OSS and vendor systems, thus giving additional insight into the dependency satate of applications.
  - Giving ownership of OSS and cloud systems to teams with software devs: When APIs are later shown to be imperfect, platform teams have software devs who can write the nontrivial migration tooling that makes the migration transparent to app teams.

### Allowing Application Developers to Operate What They Develop

- Mature DevOps aimed to simplify accountability: You build it, you run it. In practice, companies have struggled with this model. Those that have succeeded may have done so partly their platform abstractions that reduce operational complexity of understanding the underlying dependencies.
- On-call is never beloved, but engineers are generally more willing to take on operational responsibility if they're only on call for issues caused by their own apps. However, this is more challenging when operational issues caused by infra, OSS, and glue are just as prevalent as app code issues. e.g., multi-az, high-availability apps leave app teams exposed to intermittent cloud provider issues driving latenight pages; Platform engineering can build resilient abstractions that handle app failover on behalf of teams, reducing the number of these.
- When underlying systems' operational complexity is hidden behind abstractions, the complexity can be owned and operated by the platform team... But must limit supported options so that the team can provide a core set of offerings, each supporting a broad set of application use cases. Additionally, the platform must have high operational standards so that app teams can rely on them.

## Empowering Teams to Focus on Building Platforms

- Four platform-adjacent team models that _do not_ generally have the skill mix to build platforms
  - Infrastructure, focused on robust operation of underlying infrastructure. These teams have little focus on abstracting infrastructure to simplify applications, particularly across multiple infra components. In a platform engineering group, this team would balance pure infra capabilities with developer-centered simplicity.
  - DevTools, focused on dev productivity _up to_ production delivery. Little focus on solving dev productivity challenges related to systems in production running on complex infra. In a platform engineering group, these teams would banalce dev experience with production support experience.
  - DevOps, focused on app delivery to production. Little focus on ensuring their tools / automation help the widest possible audience. In a platform engineering group, these teams would balance optimal per-application glue with more general software that supports many more applications.
  - SRE, focused on reliability. Little focus on systemic issues outside reliability, often delivering impact through org practices instead of developing better systems. In a platform engineering group, these teams would balance reliability with attributes including feature agility, cost efficiency, security, and performance.
  - Note that these are outcomes driven by team missions, as contributed to by the org. Individuals from these background may assert they want to build more platforms rather than glue, but the org doesn't give them time. Redefining org expectations allows platform engineering to re-focus.

### Do platforms support innovation?

- Platforms can support org innovation by allowing for more rapid app dev through the simplified platform offering.
- However, by their nature, platforms do not support every innovation. e.g., a team may require a different data store for an innovative use cases, and they may leave the platform in order to implement. Platform builders don't fight this, nor do they try to abstract every one. Instead, the ideas may develop independently, with platform builders choosing to abstract if the idea is successful and has widespread demand.
- Thus, platforms must enable productivity and innovation within the bounds of the known, but they must also allow for teams who need to experiment with something new. The trade-off in knowing when to push teams toward centralized offerings vs when to let them experiment is a key skill for platform engineering leaders.

# Chapter 2: The Pillars of Platform Engineering

- Defn reminder: *Platform Engineering*: The discipline of developing and operating platforms with a goal to manage overall system complexity in order to deliver leverage to the business. It does this by taking a curated product approach to developing platforms as software-based abstractions that serve a broad base of application developers, operating them as foundations of the business.
- The 4 Pillars
  - Product: Taking a curated product approach
  - Development: Developing software-based abstractions
  - Breadth: Serving a broad base of application developers
  - Operations: Operating as foundations of the business

## Taking a Curated Product Approach

- Product approach: Getting out of a purely technical mindset to refocus on what customers need from your systems and their experience using these systems. Without curation, very responsive to customers but no coherent strategy, thus leading to a service center team.
- Curated approach: Not only are there specific interaction patterns and usage conventions, but you have an opinion about what is and isn't in scope for this platform and curate accordingly. Without product mindset, leads to rigid offerings for app teams that may not meet their actual needs.
- Thus, platform teams must combine both approaches for success.
- Successful implementation of Curated Product approach leads to two types of platform products:
  - Paved Paths: Layers multiple offerings together into easy-to-use workflows, successfully hiding most of the complexity. Aims for both coverage and usability: Opinionated enough to cover / encourage the common use cases(e.g., the 20% of use cases that cover 80% of needs) and focused on making these work extremely well. Requires saying "no" to outliers, allowing outlier needs to step off the path.
  - Railways: Platforms developed after the platform team has discovered a meaningful gap that isn't covered by any existing product but that would fill a need for many application teams. Discovered via a product discovery process that looks for patterns of need across app teams and investigates how teams are currently working around this missing platform. Often based on prototypes that app teams have built to meet their specific needs, generalized into a more broadly useful offering. The goal isn't only to smooth out common processes and often involves a major infra investment to bring a specific piece of functionality to the company. Examples: Batch job platform; notifications system; global app config platform.

## Developing Software-Based Abstractions

- Most common form of platform engineering today isn't at infra or dev tooling level. Rather, happening at large orgs with multiple product lines that with core, in-house capabilities that should be shared across all lines(e.g., billing). These core capabilities are extracted into a platform so that a single team can support them across all product lines. Resulting team must include software engineers, as the in-house existing capabilities will have been built by software engineers and will require ongoing maintenance. Platform may evolve to integrate SaaS products with the in-house logic, but will always require software engineers. This need for software engineers generalizes across all platform engineering teams.
- Corollary: If you don't need to write software yet, you may not be ready for "platform engineering". If a wiki pointing at approved provider offerings and instructions suffices, stick with it, as it may not be time to invest in a full platform engineering team to solve the problems.
- Be careful starting a platform engineering initiative without software engineers. Platform engineering creates leverage by using software logic developed in-house to abstract underlying - if you aren't providing that abstraction, you aren't creating platforms that manage complexity, you are vending infrastructure and passing all complexity to users.

### The Major Abstractions: Platform Service and its APIs

- Similar to SOA/microservices: Service is a major component of the platform, implementing logic to coordinate behavior the underlying OSS, vendor, and in-house systems, and exposing that logic via APIs. Without these APIs, maintained by software engineers, difficult to create abstractions that make things simpler for app devs.
- APIs can fully encapsulate the underlying(e.g., an api that abstracts Postgres away by executing queries on a sender's behalf), but determining whether that's beneficial requires looking at it from the app dev perspective. Unless you are demonstrably making engineer productivity higher, you should continue to expose access to the underlying. Making things easier for the platform team to manage is not a good enough reason to reach for full encapsulation.

### Thick Clients

- Apps may be best-served by client-side code being more complex than thin shims of pure service archs, so thick client libraries / sidecars may be useful. e.g., reliability and performance enhancements around sharding; local caching; load balancing may all be better handled by a thicker client.
- However, thick clients bring difficulties in observability, debugging, and upgrade cadence. This can favor complex logic living at the service layer.
- Worth considering both approaches in light of trade-offs.

### OSS Customizations

- Sometimes OSS is very close to providing an appropriate abstraction and just needs tailoring to the org's challenges via e.g., building custom plug-ins; customizing the OSS itself. Platform team owns building, running, maintaining this product and its customizations.

### Integrating Metadata Registries

- One opportunity of platform engineering is the ability to deal with changes / problems in the underlying on behalf of users, but this requires metadata about the underlying primitives(e.g., Who owns a service; Does the service need broad access to a given blob store; Which org should be charged for the blob store use; Who is using the blob store in a non-standard way and who will platform eng need to talk to about how to shim as we migrate to a new vendor).
- Few emerging systems to track this metadata:
  - Tag management systems: All major cloud providers provide tagging functionality and some provide tools that allow platform teams to enforce schemas and run exploration queries.
  - API/schema registries: Focused on collating compile-time information about platform and application APIs, centralizing that data for management, governance, exploration.
  - Internal developer portals(IDPs): Enhanced registries that catalog not just API and resource metadata, but platform configuration as well. Exposes a programmable UI that allows platform teams to plug in their offering, exposing as a consistent, centralized UX across all platforms.
- However, early days on those systems, unclear future: e.g., Registries can struggle when the creating teams think that users will manually populate data or that the platform team would only need to "clean up" scraped machine data. The success of these systems is likely tied to how well the registries can automatically capture and maintain metadata, as engineers will likely not take well to curating that data themselves.
- Is an IDP a required component of platform engineering offering?
  - Lots of energy around IDPs being a core offering, perhaps the most important.
  - However, this implies the highest-leverage thing that platform engineering can do today is having one place that ties together ownership of platform resources with all your platform’s self-service UIs and documentation. Integrating the majority of platform use cases that make this worthwhile is a lot of work with only "some day" potential.
  - If customers are saying that one of their biggest problems is figuring out where to go to find the right UI/documentation to use the platform then yes, may be highest leverage. If that's lower down, you may be able to just rely on wikis / docs / API links for now.

## Serving a Broad Base of Application Developers

- Target audience for platforms is app devs - not just one or two teams, a broad base of them. Sometimes app teams struggle with UX overhead of central platforms(e.g., if you're used to shell access to install tooling, having to go through platform abstraction feels burdensome). While the purpose of a platform isn't to make every single action as easy as it could be for expert users who want direct access to primitives, platform teams need to invest in the things that offer the best development and operational experience to all of its users.
- As usage broadens in terms of users and use cases, must move from developing features to also developing capabilities that make the system cheaper, safer, easier to use, including:
  - Self-service interfaces: Can't support a large set of customers without self-service access, provisioning, and configuration. If onboarding requires the platform team to do manual work or requires multiple teams to do coordinated work, the platform loses its leverage. Could be GUI or CLI tools, often integrated into CI/CD platforms. Best offerings will have an easy-to-use set of defaults for novice users while providing power users to access the building blocks themselves for advanced use cases.
  - User observability: Must have telemetry to help devs debug thei own problems throughout the full lifecycle of developing and operating applications that use the platform. Goal: "a user of the platform should be able to tell whether they're doing something wrong or the platform is doing something wrong." Unreachable ideal, but good mindset.
  - Guardrails: Can't expect your broad base of users to all be experts in the underlying systems, particularly in terms of security, compliance, reliability, cost. Misconfiguration can be costly, so key part of platform is these guardrails(default limits, protections, etc) to limit those misconfigurations, while also making sure the platform is flexible enough to react to changing primitives and biz needs.
  - Multitenancy: Platforms are only efficient if they are built to be multi-tenant(i.e., supporting different applications within the same runtime components). For internal platforms, should aim to be a central team that provides shared systems that support many applications, _not_ a team that operates one system per application. This is hard to do well in practice and again calls for both software and systems engineers on platform teams. May decide some components / customers don't fit multitenant and end up with a hybrid, but if you don't intermingle applications and/or users in at least some of the components or deployments of your platform, it's probably not a platform.

## Operating as Foundations

- Platforms must be operated as foundations: Rock-solid stability that app devs can trust their biz on. Flakiness due to your components or the underlying robs your platform of leverage, forcing your users to gain expertise reactively based on the platform's operational issues. Operating your platform at this quality level takes a lot of work and has three major aspects:

### Responsibility for the Full Platform

- Platform engineering teams must operate the full platform, not just the software developed in-house. If you're only building something that makes it easy for app devs to get started, they'll inevitably need to become experts in the underlying primitives _and_ the platform when facing production issues, thus giving up a lot of the platform's leverage. Common shapes of this:
  - Provisioning Platforms: Provision primitives but app teams take operational responsibility after provisioning.
  - Framework Platforms: Platform collates OSS and vendor library versions but app teams own all operational responsibility.
  - Tools Platforms: Platform provides tools and UIs that make it easier to manipulate OSS and vendor systems, but app teams own all operation responsibility for those systems.
  - Each of these are great parts of the plat eng team's responsibilities and in some cases it won't make sense to go beyond them, but they expose a lot of operational complexity to users, so they don't meet the larger umbrella of platform engineering.

### Supporting the Platform

- Many app teams don't deal with high volume of user support escalation whether via biz support(e.g., external services often have support staff) or because of limited users / breadth that don't generate much support need. Platform teams are different, as the broad base of use cases they cover tend to naturally lead to a steady stream of questions about edge cases, application-specific production issues, etc. Thus, user support is an expected part of platform teams.

### Operational Discipline

- Difficult to operate a system whose major functionality is based on somebody else's code(OSS, Vendor, etc). Unknown operational problems are a constant threat, must be managed by a discipline that seeks to understand and address all anomalies early, before they cause acute pain.
  - Infra-engineering culture has persisted partly because the software developers who would need to be build true platforms don't want to engage with the practices needed to operate other peoples' systems well. Platform eng teams must be proactive in their use of operational practices, understanding that they are a major part of ensuring that those platforms are a foundation the company can rely on.
- What does generative AI mean for platform engineering? Early, but some potential areas:
  - Tooling around the model experience will matter: Machine Learning Ops(MLOps) is the operations of building, training, deploying, and operating models. Similar to SDLC processes used to deliver software, but users are researchers or non-technical people instead of engineers. These non-engineer users will need coherent tooling that works for them, similar to devs staying within their IDEs, etc. Goal is to keep researchers within their familiar tools instead of forcing them to jump from platform to platform.
  - Platforms that help optimize for underlying infra efficiency will be in focus: Training costs are high, so workload placement, network costs, etc need to be optimized in a way that is transparent to the researcher.
  - Platforms developed for ML using company data will require special focus on controls and data entitlements: Changing regulation and consumer prefs will increase the need to understand data provenance, explain the outcomes of a model, and know who has access to what data. Solving these holistically across ML systems in a company will need platform solutions that meet the controls without forcing everyone into an inflexible workflow outside of their desired working context.
  - AI will help you operate better, but only if and where you have the data: ML needs data to make decisions, only your instrumentation for your platforms will feed the model. Easier with modern systems than with legacy apps and platforms.
  - Platforms will be built that curate the ecosystem of large language model (LLM) tooling for your company and ensure all of the components work together well: To date, tooling to support research and model gen has largely gone to massive companies(e.g., Google) or to understaffed data eng teams that don't have the funding/interest in operating a useable platform. Thus, many companies building in-house AI infra run into the same probs as large-scale distributed infra: handling software and hardware failures, orchestrating work efficiently, debugging when things go wrong.

# Chapter 3: How and When to Get Started

## Fostering Platform Cooperation at Small Scale

- When you first read about the outcomes of platform engineering, you think about a mature and efficient platform that enables engineers to code, build, test, and deploy with minimal friction, implying a mature and stable platform. There can be a long gap between early stage company and this maturity, but you can start the evolution. However, you will make many mistakes: Underscaling, premature optimization, evolutionary dead ends, backtracks, etc. Up to about 50 engineers, can think about the first two stages of evolution for platforms:
  - Stage 1: Ad hoc, whatever works right now
    - In small org with small team, focus is on short iterations, minimum viable product, quickest cycle time between dev and user test. Yields little formal process or tooling. Stack and tools are selected based on preference, tests may be lacking, deploys may be manual, knowledge is shared organically by the small team. This size team doesn't need a platform team, it needs the basics:
      - Source control
      - Automated CD: Use off-the-shelf tools that require minimal configuration(e.g., not k8s). Ideally, you build an artifact when you merge and it is deployed with little or no intervention(platform examples: Heroku, Netlify, etc). Balance complexity, cost, scale, lifespan. Can automate deployment and management of these via a tool that sets you up with a basis for future automation, like Terraform. Always ask whether the tool is biz-differentiating, part of the core biz and, if not, find a platform or service to do it for you.
      - Lightweight processes: Similar to Pollan - Use a process; not too much; mostly agile. Use a simple ticket system. Don't over-focus on estimates, sprints, formal agile. Consider trakcing the ratio of new feature dev to support/tech debt as a signal for velocity and an appropriate granularity level. Don't do more, including rushing to create platforms, as you likely don't have the people to dedicate to it and it'll risk impacting rapid iteration.
  - Stage 2: Somewhat managed, taking a more principled approach
    - Larger team brings more complex communication requirements, more complexity in writing, reviewing, merging, deploying. At this stage, platform components emerge, but you still shouldn't launch a platform team - you likely have a core of people focused on infra and automation who may serve as the foundation of a future platform team, but for now it should remain a shared responsibility.
    - You will start to outgrow some initial tool choices that worked for 5-25 devs due to desire for better performance, capabilities, cost. Some decisions that were made based on dev preference and require niche knowledge or skills may also need to be rethought.
    - Focus remains on whether the tooling/sln choices you make are core to the biz and, if not, outsourcing them.
    - This is the only stage during which you should entertain "Should we rewrite it in...".
    - Recommendations for this stage:
      - Local Development: Good stage to start automating your local dev env. Start with shell scripts around a container tool, which will be hard to manage, suffer from config drift, and will break during upgrades / updates. Mitigate by colocating dev env setup with source code; publishing container images with the elements of localdev stack when you build deployables, allowing the shell scripts to simply install container tooling, download images, launch them. Implement git hooks to prompt/update localdev env to ensure that devs stay up-to-date.
    - All the world's a stage: Consider more robust testing / deployment processes, as team growth will likely make pushing directly to production riskier.
      - More robust tests, CI: Test coverage metrics may set a good cultural norm of adding tests as teams add code, even though they're potentially a flawed metric. Good time to start an environment heat map that tracks most frequent areas of bugs, especially customer-facing ones, as these are good areas for test investment.
      - Branch-based deploys: Build slns that allow eng to deploy from branches, either as part of local development or for deployment to on-demand or ephemeral envs.
      - Feature flagging: Don't build it yourself as it's not a core biz focus, but do start using a feature flag platform.
    - Observability: Have basic observability and workflows - Build fail alerts, env performance, logging, etc.
    - Have the right tech: All of the above will require automated infra. Build it out to fully manage Prod, then extend it to manage CI and ephemeral envs.
    - Socialize change and decision making: Consider more formal change management processes for things like your env stack and arch. Decisions have a much wider impact, so individual pref must yield to org needs. e.g., RFC process, ADRs, etc.
    - Knowing when to move up: Hard to tell, often the first indication of stress is when a tool or process breaks and leads to a cascade.

## Creating the Platform Teams that Replace Cooperation

- Platform starts coming into focus as team grows: Too much tech debt causing low productivity or reliability issues; biz change that exposes unmanageable complexity; etc.
- Rule of thumb is Dunbar's number, the piont at which no person will know all of the others, so processes that define owners / accountability become important.
- In platform team's case, often difficult up front as it removes old cooperate patterns and creates conflict in terms of whether the platform team will do the work to fix the platform or the app teams will do the work to fix how they use the platform.

### Are the Benefits of Centralizing Ownership Worth the Cost

- Natural to want to centralize when you see app teams building similar stuff, but centralization adds a new coordination point for the teams that can slow them down. Final decision should be based on leverage rather than only efficiency: the possibility of two teams building the same thing doesn't meet the standard. e.g., Billing must be consistent, on-time, reliable, so could be a good centralization point. Caching, web frameworks may not.
- Rely on Total Cost of Ownership: If the system needs a large team to build and maintain it, and it can support multiple teams without complex configuration or logic changes per team, it may be a good centralization point. If it has small cost to build and maintain, especially if apps will likely need to configure or extend it, not a good candidate.

### Realize the Collective Dynamic is Gone

- Customers/coworkers will have trouble breaking out of old, cooperative habits in favor of more process. Platform team members will, too, thinking they can roll out changes in a way / timeframe that were fine with 50 users but may not be for 500. Breadth of use cases has increased, and platform team will no longer have a broadly representative set of perspectives from a small number of users. Change will be slower.

### Focus on Solving Problems, Not New Technology or Architecture

- Since platform team is formed only when there's clear leverage for the org, starting state will be disparate with poor API and service boundaries, if there are any at all(e.g., common code, libraries). Initial user asks will be to make these easier to use, and platform team's initial instinct will be to step back and look for a new arch or technology. However, change is going to be slow and platform migrations can take years during which applications won't see help with pressing needs as resources are dedicated to the emergent platform. Initial goal of platform team is to build trust: Figure out how to maintain collaboration with the org; deliver app dev value quickly to dispel skepticism; detangle things more than rearchitect with a focus on incremental value.

### Beware of New Engineers Coming from Much Bigger Companies

- EMs and senior eng from big tech companies will have experience with the scale you want, but may not have experience with the scale you're at(and have been at). These people may be very confident in that scaled solution being the fix to problems, but they may not know the cultural differences that face your org. Interview these people for their skills and attitude, not just their previous gig.

### Be Slow to Hire Product Managers(and Avoid Project Managers)

- Platform team model should already be working and fully staffed before PdMs:
  - Hiring non-engineers before delivering value as a small team suggests that the team is poor at communicating with its users.
  - The first year or two of platform building is often straightforward problem-solving and the number of customer teams is generally small enough that engineers/EM can gather lightweight requirements.
  - Early Sr Eng and EMs need to establish communication culture. If you bring PdM in, they will own that culture, and engineers will fail to develop customer empathy.
- Rule of thumb: PdMs should be between the number of team level managers and the number of managers of managers. For PjMs, shouldn't need til 50 people on platform team, and 1:50 is a good ratio - needing one earlier is a sign that you need to better define your platform's abstractions to avoid its changes requiring so much user work.
- Continue to be careful during interviews for process fit - many PdMs and PjMs who are great working within well-defined guardrails of larger orgs may struggle with less process and spend their initial time trying to re-create them.

### Bonus Problems for Integration / Shared Services Platforms

- Platform team that horizontally supports multiple _external_ products e.g.: Billing; User login, identity, access management; Mobile and web app platforms; Revenue systems; Shared app services(notifications; search; analytics; messaging).
- These teams serve external customers and have external visibility. Questions to ask before forming:
  - How will you partner with external product management?
    - Biz-facing products will include Biz-facing PdMs, so hiring a PdM earlier makes sense... but not every successful biz-focused PdM will be successful as an internal platform PdM, as they're used to being eval'd based on revenue/profit delivery instead of reliability and developer productivity. Additionally, the importance of things like UI design and other visible elements is higher in customer-visible components, while the true value of the platform is in its core tech and internal customer usability.
    - How will everyone find your offerings? Platform discoverability at a large company is difficult, particularly when the platform solutions are semi-optional and not needed by all. e.g., You may not want to mandate that all teams use the search platform, but it can cause friction when not using the platform causes inconsistencies for customers using different biz systems. Particularly difficult due to the tendency for platform teams to give funny names to their platforms("glengarry" vs "billing platform"). Plan out how you will solve discoverability up front: Use a name that makes sense; Write docs that are easily searchable; send email announcements; run learning sessions; etc.
    - Being stuck in the middle: Integration platform teams are generally not part of the core/infra platform org, so may not have superuser access, which can make building shared platforms challenging. These teams are also subject to errors on both sides of the stack, challenges in debugging and operations. Key to keep these teams aligned with other platform teams and to be mindful of conflict potential, especially when initiatives span the two.

## Tranforming a Traditional Infrastructure Org

### Your Whole Engineering Culture Has to Change

- Infra orgs are great at cost management, vendor negotiations, running systems at scale. Include specialists on databases, networking, kernel issues. Generally not great at user empathy, as their users are a captive audience. A shift must happen in order to facilitate a curated product approach.

### Identify the Most Promising Areas to Start

- Don't do it all at once, as the most conservative parts of the org will distract you from making the readier teams successful. Instead, focus on teams delivering modern offerings, who may have the elements needed to make the transition: Teams including software engineers; bespoke software with a high rate of change; teams that most need modernization(e.g., physical infra to virtual to container-based). Teams that mix software eng with infra eng are a good place to start.

### Recognize that you Can't Just Rub Product Managers on it and Call it a Day

- PdMs alone can't ensure success. Eng teams must feel ownership for delivering a great product to their customers, and PdMs will have difficulty closing that gap for the team. Start with teams that are customer-centric and introduce some product-oriented approaches to them.

### Change the Way You Support Your Products

- Ticket system can make customers feel more like a burden than a focus. Keep a close eye on how you provide support, your response time to questions, and how you triage. Engineers should spend time supporting their products and should be regularly answering questions to feel their users' pain. Do not make this optional or delegate to junior engineers; senior engineers must build the skills to provide humane products and interact with users in a helpful way. Engineers who can't be trusted to productively support, enable, and help users is unlikely to build user-friendly products. Those products are at risk of future rewrites due to high support load / complaint load.

### Update your Interview Process

- Add "Customer empathy screening", specifically asking questions on how interviewees think about writing user-friendly code. Set the tone that you expect your developers to think not just about how to build the systems, but also about the people who are going to use them or work with them.

### Update Your Systems of Recognition and Reward

- Important to recognize not only engineers who solve big technical problems, but also engineers who smooth out usability edges, interact well with users, and prioritize work on users' behalf. Celebrate and compensate both, and make sure engineering ladder reflects both.

### Don't Have Too Many Project Managers

- Project manager dependent infra orgs may lack planning skills around a very common infra need: migrations. If migrations are so painful to both platform teams and customer teams that you need PjMs to understand all of the dependencies and to track what's happening, reflects poor ownership of your software's UX(i.e., migrations are part of your UX). If you frequently expect users to do all of the work to migrate to new systems on the platform team's timeline, failing to consider their obligations or needs, you have a lot of work to do. Part of moving to a platform model is owning much more of the migration, as the platform value add must include lowering the migration pain for users. Limiting PjMs now forces platform engineers to do the work, which will help the good ones realize that if they automate then the tedious project management piece will be lower: Saving platform team time saves customer time. However, must make sure engineers have time to do the work.

### Accept That Your Team Will Spend More Time Talking to Customers and Less Time Writing Code

- No shortcut to product mindset transition, nor can a PdM or customer advisory board fill the gap. Team must spend more time with users, more time strategically planning to address holistic concerns. Team will need to change how they work: fewer tickets completed vs pure ticket ops.

### Do the Necessary Restructuring

- Some people will not fit in the new culture, including leadership. You may need to replace them.

### Keep It Fun!

- By the time you get to this transition, there's often an us-versus-them mentality between infra and app eng. Try to make it fun: Get feedback from users about what they love; share kudos as they come in, celebrate improvements in customer satisfaction metrics, and make sure to include platform teams in celebrations where their work enables an app team. Be positive. Similar to devops transition, where engineers learn not to throw their code over the wall to an ops team.
