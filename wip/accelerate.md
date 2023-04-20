# Accelerate

## Chapter 1

### Intro

Accelerate:
- How quickly we deliver to customers
- Engagement with customers to get feedback and understand demand
- Anticipation of compliance / regulatory changes that will impact
- Risk response to emerging threats, economic or security

Recent study: "Strategic use of technology explains revenue and productivity gains more than M&A and entrepreneurship"

*DevOps* How to build secure, resilient, rapidly evolving distributed systems at scale.
- e.g., CI/CD, lean, etc.
- Report: Leadership tends to over-estimate their progress in comparison to those actually doing the work.

### Focus on Capabilities, Not Maturity

Why Capabilities?
- Maturity models focus on arriving at a mature state and then declaring themselves done. Capabilities focus on continuous improvement.
- Maturity models prescribe a similar set of tech, tools, or capabilities for every set of teams and org. Capability models are multidimensional - Different parts of the org can take a customized approach, focusing on capabilities that will give them the most benefit based on their goals.
- Capability models focus on key outcomes and how the capabilities can drive those outcomes. Maturity models generally measure the tech proficiency / tooling install base in the org without measuring outcomes, so they don't really reflect biz impact.
- Maturity models define a static level of technological, process, and org abilities to achieve rather than taking into account the shifting landscape. Capabilities allow adjustment, focus on developing skills and capabilities needed to remain competitive.

### Evidence-Based Transformations Focus on Key Capabilities

- What to focus on? Product vendors: Whatever supports the product offering. Consultants: Whatever they're selling and using to assess success. Too prone to "loud voices", not evidence-based.

Things that don't predict performance
- Age and tech used for the app(old Java vs greenfield)
- Whether ops or devs performed deployments
- Change approval board reqs

### The Value of Adopting DevOps

Why does it matter? High performers deploy 46x more frequently; 440x faster lead time from commit to deploy; 170x faster mean time to recover; 80% less likely for change to fail

## Chapter 2: Measuring Performance

### The Flaws in Previous Attempts to Measure Performance

Most focus on _productivity_, thus focusing on outputs rather than outcomes, individual or local measures rather than team or global. e.g.:
- Lines of code: Prone to inflation or unreadability, etc
- Velocity: intended to allow capacity planning, but misused to measure team productivity or compare teams. Velocity is relative and team-dependent based on different contexts. Teams quickly learn to game the system: Inflated estimates, complete tix ASAP rather than collaborate. 
- Utilization: Works to a point. Once there is no space capacity(Slack) to absorb unplanned work / changes, lead times go up.

### Measuring Software Delivery Performance

Must focus on global outcomes so that teams aren't pitted against each other(e.g., ops vs eng); focus on outcomes not output - don't reward busywork that doesn't contribute to org goals.

Four Measures

#### Delivery Lead Time: How long it takes to go from customer request to customer satisfaction.

In product dev, where goal is to satisfy many customers in ways they may not expect, two parts: The time to design and validate a product / feature; the time to deliver that feature to customers. 

Product Design: often has high variability, hard to pin down when clock should start. e.g., Create new products / services using hypothesis-driven delivery, modern UX, design thinking; Feature design and implementation may require work that has no precedent; Estimates are uncertain; Outcomes are variable.

Delivery: Enable fast flow from dev -> prod, reliable releases, smaller batches; Integration, test, deployment must be done rapidly; Cycle times are predictable; Outcomes have low variability.

Shorter Delivery lead times enable fast feedback, incident response.

#### Batch Size(Deployment Frequency): How often teams deploy to Prod / App Store / etc

Lower batch size => lower risk, higher efficiency / motivation / urgency, less chance of schedule growth. Batch size is hard to measure in software, so use deploy freq instead. 

#### Mean Time to Recovery: How quickly can we recover from failure?

Used instead of time between failures because bugs are inevitable.

#### Change Failure Rate: How often changes to the product cause degraded service, rollback or need a hotfix.

#### Supporting Data

High performers deploy on demand(multiple times per day); Have < 1hr lead time; Have < 1hr MTTR; 0-15% CFR. Each of these metrics is _worse_ for lower performers - No trade-offs between them.

Beware trying to increase tempo without resolving underlying issues. This may worsen your metrics.

Note that Medium Performers may have _worse_ CFR performance than Low in some instances. Maybe: they're mid-transition, trying to overcome quickly, leading to more frequent rework.

### The Impact of Delivery Performance on Org Performance

Org perf measured on profitability; market share; productivity, which have proven highly correlated with ROI. High Performers 2x likely to exceed these goals as Low. Also 2x as likely in quantity of G&S, operating efficiency, csat, quality of products / services, achieving org mission.

Experiment-based approach to prod dev <=> technical practices of CD. Don't outsource strategic / biz-crit software: Needs to be super efficient. DO buy / acquire software that's not directly biz-crit / strategic. Understanding which is which is key - Use Wardley maps.

### Driving Change

The measures listed allow comparison of teams, allow us to start experimenting and measuring how tweaks help / hurt. Be careful - In dysfunctional orgs, measurement will be used to control => employees will hide info that challenges existing rules, strats, power structures. 

## Chapter 3: Measuring and Changing Culture

### Modeling and Measuring Culture

Three levels of culture
- Basic assumptions: Formed over time as members of a group / org understand relationships, events, activities. These are least visible - "Just known", hard to articulate.
- Values: More visible. Lens through which the group interprets relationships, events, activities. May drive social norms.
- Artifacts: Most visible. Mission statements, creeds, procedures, heroes, rituals.

Westrum Typology of Org Culture
- Pathological: Power-oriented. Lots of fear and threat. Info is hoarded / withheld for personal gain / politics.
- Bureaucratic: Rule-oriented. Protect departments, turf. Departments do things by their own books.
- Generative: Performance-oriented. Focus on mission. How do we accomplish our goal? Continually ensure we're aligned.

The org culture predicts how info will flow. "Good" information will provide answers to the asker; be delivered quickly; be understandable to the receiver. Also predicts performance. See https://share.getcloudapp.com/d5uv28lD

### Measuring Culture

Analyze data using statistically valid questionnaires, likert scales. If analysis bears out, we have a construct that can be used in further research. Constructs must be judged on discriminant validity(items that shouldn't be related are unrelated); convergent validity(items that should be related are); reliability(items are interpreted consistently). See questions: https://share.getcloudapp.com/NQuDBEZg

Generative culture enables info processing
- People collab effectively and trust is hierarchically horizontal and vertical.
- Emphasizes the mission, so people put aside their personal / departmental issues to focus on mission.
- Encourages a level playing field where hierarchy isn't important.

Bureaucracy isn't necessarily bad - Should ensure fairness by applying rules to administrative behavior. Rules would be same for all, derived from accumulated knowledge by experts in their fields. Becomes bad when following the rules becomes more important than following the mission.

### What does Westrum Org Predict?

Orgs with better info flow are more effective.
- Trust and co-op between people across the org.
- Higher quality decision making. Info is widely available to make decisions and they are easily reversed because teams are transparent.
- More likely to retain happy people since problems are surfaced and dealt with.

### Consequences of Westrum's Theory

Resilient and ability to innovate are essential in the face of tech and economic shifts.

Delivery performance as a construct: Only 3 of the measures are statistically valid for software delivery performance. CFR is highly correlated, but doesn't pass test. The 4 together are still good for categorizing teams as H / M / L.

Google Research: Looking for common factors of high-performing teams. Found that team members' traits are less important than how they interact, structure their work, view their contributions.

How does org deal with failure? Pathological: Throat to choke, even though a single person is almost never at fault. Human error is the start of the investigation, "How could we improve info flow so that people have better info or tools to prevent catastrophic failure?"

### How do we change culture?

Lean practices + CD positively impact Westrum org culture. 

## Chapter 4: Technical Practices

### What is CD?

Set of capabilities that allow safe, quick, sustainable deployment to production.

- Build quality in: Build a culture supported by tools and people where we can detect issues quickly and fix them immediately.
- Work in small batches: Small, deliverable chunks of measurable biz outcomes let us get feedback early and course correct. There is some overhead, but we avoid the wrong work.
- Computers perform repetitive tasks; people solve problems: Invest time in automating regression testing and deployment so that people can do the rest.
- Relentlessly pursue continuous improvement: High performing teams are never satisfied: always trying to improve. They build it into their everyday work.
- Everyone is responsible: Bureaucratic orgs tend to favor vertical slices: Ops focuses on ops; testing on quality; dev on throughput. Reality is that all are system-level outcomes. Mgmt must make the state of these outcomes transparent with measurable, achievable, time-bound goals; then help teams work toward the goals.

To implement

- Comprehensive config mgmt: Must be able to provision envs and build, test, deploy software fully automated using data from VCS. Changes must be applied using automation and data from VCS. 
- CI: Branch integration is expensive. Branches should be short-lived(<= 1d work), integrated frequently. Changes trigger build + tests, failures are fixed immediately.
- Continuous testing: Testing doesn't start once we're dev complete, should happen throughout. Automated unit + acceptance tests on commit. Tests must run locally. QA should perform exploratory.

### The Impact of CD

Capabilities
- The use of version control for application code, system configuration, application configuration, and build and configuration scripts
- Comprehensive test automation that is reliable, easy to fix, and runs regularly
- Deployment automation
- Continuous integration
- Shifting left on security: bringing security—and security teams—in process with software delivery rather than as a downstream phase
- Using trunk-based development as opposed to long-lived feature branches
- Effective test data management
- A loosely coupled, well-encapsulated architecture
- Teams that can choose their own tools based on what is best for the users of those tools

Questions
- Our application code is in a version control system.
- Our system configurations are in a version control system.
- Our application configurations are in a version control system.
- Our scripts for automating build and configuration are in a version control system.

As expected, these correlate with performance, but also with decreased deployment pain and burnout. Implementing these also improve culture by giving devs tools to detect problems early, time and resources to learn and develop, and authority to fix problems immediately.

Desired outcomes, which are impacted by above capabilities

- Teams can deploy to production (or to end users) on demand, throughout the software delivery lifecycle.
- Fast feedback on the quality and deployability of the system is available to everyone on the team, and people make acting on this feedback their highest priority.

Org outcomes, also linked to CD
- Strong identification with the organization you work for
- Higher levels of software delivery performance (lead time, deploy frequency, time to restore service)
- Lower change fail rates
- A generative, performance-oriented culture

Does CD help quality?

- Change failure rate
- The quality and performance of applications, as perceived by those working on them
- The percentage of time spent on rework or unplanned work
- The percentage of time spent working on defects identified by end users

CD strongly correlates with % of time spent on rework/unplanned work, emergency deploys/patches, urgent doc requests, etc. Unplanned work forces an urgent stoppage / drop-everything effort, so is particularly hard.

### CD Practices: What Works and What Doesn't

- Version control: Keeping system and app config in VCS more strongly correlated with IT perf than app code. 
- Test automation
  - Reliable tests: Tests should pass and instill confidence. Failures should indicate a real problem. Quarantine "bad" tests - Delete if appropriate.
  - Devs create and maintain acceptance tests, which run locally. QA-maintained tests are not correlated with performance. _Might_ be because devs who create and maintain tests write more testable code and have skin in the game on their tests.
  - Tests must auto-run on commit.
  - New builds must be availble for testers to perform exploratory.
- Test Data Management: Sufficient test data exists and doesn't limit their automated testing.
- Trunk-based Dev: Avoid long-lived branches.
  - Great teams have <= 3 active branches with lifetimes <= 1d before merge to trunk. These results were repeatable across team size, org size, industry.
  - What about git flow? As long as branches are <= 1d, ok.
- Information security: High-perf teams integrated security into the delivery process. Infosec involved at every stage of lifecycle, from design thru demos and automation, but were involved in such a way that they didn't block dev - Just part of dailly work.

## Chapter 5: Architecture

### Types of Systems and Delivery Performance

Greenfield, end-user systems, outsourced custom dev, mainframe, etc: Low performers were more likely to be using outsourced software. Also, low performers more likely on mainframe. Otherwise, no correlation btwn system type and perf. Focus should be on arch _characteristics_ rather than implementation details - A legacy system can achieve characteristics, and a micro-svc arch can miss them.

### Focus on Deployability and Testability

High performers more likely to agree
- Testability: We can do most of our testing without requiring an integrated env(an env where multiple svcs are deployed e.g., Staging or ODE)
- Deployability: We can deploy / release our app independently of dependent services

So, does a loosely coupled arch drive perf? Yes, teams should be able to
- Make large-scale changes in svc design without external permission
- Make large-scale changes in svc design without depending on other teams to make changes on their end
- Complete work without coordinating with external teams
- Deploy / release on-demand w/o regard to external svcs
- Test on demand without requiring an integrated test env
- Perform deployments during biz hrs with negligible downtime

Requires teams to be x-func, from dev thru prod(including ops in prod, observability). i.e., Conway's law: "orgs which design systems... are constrained to produce designs which are copies of the comm strucutre of these orgs". Evolve teams to avoid this([Inverse Conway](https://www.thoughtworks.com/radar/techniques/inverse-conway-maneuver)) - Still want inter-team collab, but never want fine-grained decisions blocked during implementation. Use bounded contexts, APIs, test doubles, virtualization, but keep the goal in mind. SOA without meeting these goals won't drive high perf(see [Yegge's Rant](https://gist.github.com/jezhumble/a8b3cbb4ea20139582fa8ffc9d791fb2)).

### A Loosely Coupled Architecture Enables Scaling

Allows better delivery performance, increasing tempo and stability while reducing burnout. Allows team growth with linear-or-better productivity gains.

Commonly thought that adding a dev to a team will increase team productivity but reduce individual due to comm / integration overhead. Based on *deploys per dev per day*, as we add devs: Low performers deploy less frequently; Medium performers remain constant; High performers deploy more. 

### Allow Teams to Choose Their Own Tools

A tools list enables reduced env complexity; ensures skills are avail for maintenance throughout lifecycle; increases purchasing power; ensures licensing. (ed: also, security, etc). *But* limits teams from choosing the most suitable tools and experimenting with new approaches. Teams that decide which tools to use perform better.

Where to standardize? Architecture, configuration, infrastructure. Security should be pushing pre-approved, easy-to-use libraries, packages, toolchains, processes to prevent blocking teams. Building great internal tools with a clear focus on customer sat(internal or not) encourages team uptake of those tools. Teams that choose otherwise are sending a powerful signal.

### Architects Should Focus on Engineers and Outcomes, not Tools or Tech

Don't focus on microsvcs vs serverless; k8s vs mesos; CI platform; etc. Focus on collaborating with engineers to help them achieve better outcomes, provide the tools to enable those outcomes.

## Chapter 6: Integrating Infosec Into the Delivery Lifecycle

Infosec is generally lightly staffed, but devs are generally ignorant of even common risks. Building sec in improves sec quality, resulting in less remediation time.

### Shifting Left on Security

- Security reviews for all major features, performed in such a way that they don't slow down dev.
- Infosec integrated into entire lifecycle, from dev through ops - They should contribute to design, give feedback on demos, ensure that test automation covers security cases. 
- Easy access to the right patterns: Pre-approved, easy-to-use libraries, packages, toolchains, processes to follow.

Goal: Give devs the means to bake-in security so that security reviews are simpler - Reviewing a full system after-the-fact is much more difficult. Sec won't be able to keep up as deploys accelerate, meaning they are bound to be the bottleneck if not addressed. Result: High performers spend 50% less time remediating security issues than low performers.

### The Rugged Movement(or DevSecOps)

tl;dr: Security is everyone's responsibility.

Manifesto

- I am rugged and so is my code
- Software is a foundation of our modern world
- Great responsibility with the role of building software
- Code will be used in ways I didn't expect or design for, for a longer lifetime than I expect
- Code will be attacked by talented adversaries
- I will not be a a source of vulns
- My code will support its mission
- My code can face challenges and persist

## Chapter 7: Management Practices for Software

### Lean Management Practices

Core components

- Limited WIP: Using limits to drive process improvement, increase throughput
  - Ensure that teams aren't overburdenend, leading to longer lead times
  - Expose process obstacles
  - Doesn't srongly predict software delivery performance except when combined with visual management + prod feedback
  - Only good if ltd WIP is used to expose obstacles and those obstacles lead to process improvement(thus +throughput).
- Visual Management: Creating and maintaining widely available visual displays of quality and productivity metrics(must be aligned with biz goals); current work(+ defects).
  - Used to share information(e.g., kanban board or storyboards)
  - Include data on quality and productivity; defect rates and failures.
  - Must be widely accessible
- Feedback from Prod: Using app performance data + infra monitoring to make biz decisions on a _daily_ basis
- Lightweight Change Approvals

First three are tied to -burnout, +generative culture

### Implement a Lightweight Change Management Process

- Scenarios
  - All Prod changes must be approved by an external body(mgr, change advisory board, etc) -> Lower delivery performance
    - Does it help with stability? No - No correlation with change fail rate, higher lead and restore time, fewer deploys
  - Only high-risk changes(e.g., database) require approval -> No correlation
  - Only peer review required -> Higher delivery performance
  - No approval process -> Higher delivery performance
- Recommendation: Peer review-based approval system(intra-team reviews or pair programming) + deployment pipeline to detect and reject faults.
- What about Segregation of Duties(e.g., for PCI)? No requirement for external change board / ops approval. Instead: Before or just after commit, require non-author on same team to record approval; and only deploy to prod using a fully automated pipeline(thus committed, validated in CI pipeline) - CI pipeline provides change log.
- Is there a role for external governance? Yes: Governance teams should be monitoring delivery performance and helping teams improve it by implementing practices to increase stability, quality, speed(CD, Lean, etc).

## Chapter 8: Product Development

Agile has won, but lots of teams following some practices but not addressing org culture or proceses: Months up front for budgeting, analyzing, requirements gathering before starting work; big batches of work with infrequent releases; low or no customer feedback loop.

### Lean Product Development Practices

- Four capabilities
  - Small batches: Products and features are sliced into small batches(<1 week), released frequently. MVPs(prototypes with just enough features to enable learning about the product and its biz model) are embraced.
    - Work broken into features that allow for rapid iteration instead of infrequently released, complex features.
    - Allows for tighter feedback loop(e.g., via A/B testing).
    - Ability to experiment quickly is highly correlated with CD practices
  - Flow of work is visible: Teams understand and have visibility into the flow of work from the biz down to the customers, including status of products and features
  - Feedback: Orgs seek out customer feedback and incorporate it into design - Customer satisfaction metrics; feedback on quality and features.
  - Experimentation: Dev teams have authority to create and change specs during dev without requiring approval
- These four predicted higher software delivery and org performance, better org culture, lower burnout
- Virtuous cycle: Software delivery performance predicts Lean practices - Improving delivery effectiveness improves ability to work in small batches and incorporate customer feedback

### Team Experimentation

- Teams that claim Agile but are required to follow requirements created by different teams may result in products that are less engaging and don't deliver expected results
- Gathering feedback even during early stages allows to modify product requirements as new ones emerge. Teams that can't change requirements without external approval are limited in their ability to innovate.
- Experimentation predicts profitability, productivity, market share
- Not total freedom - Experimentation must be combined with other three capabilities to help team make informed choices about design, dev, and delivery of work. Visibility boards keep people informed. 

### Effective Product Management Drives Performance

- Virtuous cycle: Lean product managements -> better delivery + generative culture + lower burnout; Better software delivery -> More Lean product management.
- Small batches crucial to enable tight customer feedback that drives better products

## Chapter 9: Making Work Sustainable

### Deployment Pain

- Defn: The amount of fear and anxiety teams feel when they push to prod. "Are deployments feared, disruptive to work? Or are they easy and pain-free?" e.g., off-hours deploys
- Intersection of development and ops
- Painful deployments predict poor delivery, org performance, culture
- MS Example: Pre-devops, engineers reported work/life balance satisfaction of 38%; After practices 75% - Better able to balance duties during work hours, no manual deploys, less take-home stress.
- Teams must be aware of full deployment process - Otherwise they can't see downstream consequences of their work.
- Capabilities that drive lower deployment pain: Test + Deployment automation; CI + trunk-based dev; left-shifted security; test data management; loosely coupled arch; ability for teams to work independently; version control of everything required to reproduce prod environment. (See ch 4,5)
- Principles of better deploys
  - Software written with deployability in mind: Expect multiple environments; Detect and tolerate env failures; Independently updatable components. Useful error messaging when something goes wrong.
  - State of Prod systems can be repro'd(w/o data) via automation: Manual changes lead to failures due to typing, copy/paste, out-of-date docs; Config drift leads to additional deploy time work as operators try to make sense of differences.
  - Build intelligence into the app and platform: Ideally, enable one-command/click deploy.

### Burnout

- Physical,  mental, emotional exhaustion from overwork or stress leading to apathy, feelings of helplessness, low productivity, cynicism, inability to take sense of accomplishment from work, etc.
- Stress may be as bad for health as secondhand smoke, obesity. May cost companies in sick time, turnover, long-term disability.
- Reduce burnout by teaching leadership to concentrate on: Fostering a respectful, supportive, blameless work env; Communicating a strong sense of purpose; investing in employee development; Asking employees what is preventing them from achieving goals, then fixing those blockers; Giving employees time and resources to experiment and learn; giving employees authority to make decisions that affect their work and jobs, especially in areas where they are responsible for outcomes.
- Leiter and Maslach research
  - Common burnout problems
    - Work overload: job demands exceed human limits.
    - Lack of control: inability to influence decisions that affect your job.
    - Insufficient rewards: insufficient financial, institutional, or social rewards.
    - Breakdown of community: unsupportive workplace environment.
    - Absence of fairness: lack of fairness in decision-making processes.
    - Value conflicts: mismatch in organizational values and the individual’s values.
  - Many orgs try to fix the person but not the environment, but research indicates the inverse has higher likelihood of success
- Measuring burnout: "Do you feel burned out or exhausted?" + "Do you feel indifferent or cynical about your work or do you feel ineffective?" + "Does your work have a negative effect on your life?"
- Highly correlated factors for burnout
  - Pathological, power-drive org culture: Seek supportive, respectful, blame-free environment.
  - Deployment pain: Complex off-hours deploys - Ask teams where it hurts, fix it.
  - Effectiveness of leaders: Limiting WIP and eliminating roadblocks.
  - Org investment in devops: Training, support, and time to acquire new skills
  - Org performance: Experimentation-driven culture that supports failure, learning, autonomy. Space to do creative, value-add work during work hours(e.g., 20% time to work on new projects)
- Org values <-> Individual values - Misalignment -> bunrnout. e.g., employees are environmentally conscious but org dumps waste into rivers.
- Org values are those felt by employees vs stated values. 

## Chapter 10: Employee Satisfaction, Identity, and Engagement

### Employee Loyalty

- High performers have better eNPS(employee Net Promoter Score); are 2.2x more likely to recommend their org as a great place to work. Studies show this correlates with better biz outcomes.

### Measuring NPS

- True NPS-> Single Q, 0-10 scale: "How likely is it that you would recommend our company/product/service to a friend or colleague?"
  - Promoters(9-10): Create greater value for the company as they buy more, cost less to acquire, stay longer, generate positive word of mouth.
  - Passives(7-8): Satisfied but not enthusiastic. Less likely to refer, more likely to leave.
  - Detractors(0-6): Expensive to acquire and retain, more likely to leave and hurt biz through negative word of mouth.
  - NPS Score = % Promoters - % Detractors
- In Accelerate, 2 questions
  - Would you recommend your ORG as a place to work to a friend or colleague?
  - Would you recommend your TEAM as a place to work to a friend or colleague?
- Employees of high-performing teams are 2.2x more likely to recommend their _org_ to a friend as a great place to work; 1.8x  more likely to recommend their team.
- Research: "Companies with highly engaged workers grew revenue by 2.5x as much as those with low engagement. Publicly traded stocks of companies with a high-trust work environment outperformed market indexes by 3x from 1977 through 211."
- Correlates with biz outcomes:
  - The extent to which the org collects customer feedback and uses it to inform the design of products and features
  - The ability of teams to visualize and understand the flow of products or features through development to the customer
  - The extent to which employees identify with the org's values and goals, and the effort they're willing to put in to make the org successful.
  
### Changing Org Culture and Identity

- Orgs that don't treat employees like expendable resources - invest in them, enable them(e.g., continuous delivery) - get higher performance and productivity.
- How much do employees identify with the org? 6 questions, Likert scale(Disagreement <-> Agreement):
  - I am glad I chose to work for this organization rather than another company.
  - I talk of this organization to my friends as a great company to work for.
  - I am willing to put in a great deal of effort beyond what is normally expected to help my organization be successful.
  - I find that my values and my organization’s values are very similar.
  - In general, the people employed by my organization are working toward the same goal.
  - I feel that my organization cares about me.
- Teams implementing CD and with an experiment-oriented approach to product development build better products and are more connected to the org -> Virtuous cycle in which higher delivery performance increases how quickly teams can validate ideas, leading to higher job satisfaction and org perf.
- Identity includes value alignment with team and org goals - Mismatches here drive burnout, so increased indentity reduces burnout by aligning personal and org values. - Normal, less effective flow: Reqs handed down to dev teams -> Devs deliver work in large chunks, low sense of product control / customer outcomes / connection to org.

### How Does Job Satisfaction Impact Org Performance?
