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
