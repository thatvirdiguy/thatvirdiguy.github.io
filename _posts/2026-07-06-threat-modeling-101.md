Welcome to Threat Modelling 101.

The title is intentionally reminiscent of a university course. While there won't actually be an exam at the end, the goal is the same: to build a solid foundation that you can carry into real-world software and security work.

Whether you're completely new to threat modelling or have heard terms like STRIDE and attack surface thrown around without fully understanding them, this guide is designed to provide a practical introduction. By the end, you should have a clear understanding of what threat modelling is, why it matters, and how to approach it in a structured way.

We'll begin by answering the most fundamental question: What is threat modelling? Along the way, we'll look at a few commonly accepted definitions and discuss why threat modelling is an essential activity during the software development lifecycle – and why its value extends well beyond development itself. From there, we'll introduce some of the terminology you'll encounter during a threat modelling exercise. Understanding concepts such as threats, vulnerabilities, assets, risks, and mitigations makes it much easier to follow discussions and contribute effectively. Next, we'll explore some of the most widely used threat modelling methodologies, including STRIDE, and discuss where each approach fits. Finally, we'll walk through a practical threat modelling process based on a combination of established methodologies, before bringing everything together with a simple, end-to-end example.

### What is Threat Modeling?

If you need a textbook definition, it's something along the lines of: Threat modelling is a systematic approach to identifying, assessing, and mitigating security threats.

It's accurate, but it isn't particularly useful.

A more practical way to think about threat modelling is this: Threat modelling is the process of looking at a system from an attacker's perspective and asking, "What could go wrong?"

That's really all there is to it.

Interestingly, this isn't a concept that's limited to cybersecurity or software development. We perform threat modelling in everyday life without even realizing it. Imagine you're building a house. You install locks on the doors because doors are an obvious entry point into your home. If you live in an area where, unfortunately, break-ins are common, you might also install CCTV cameras or an intruder alarm. In each case, you're identifying ways someone could gain unauthorized access and then putting controls in place to reduce that risk. That's threat modelling. You're examining your system – in this case, your house – and asking questions like:

- Can someone, who's not supposed to be here, come in?
- What could they do if they did?
- What can I do to prevent it?

Software systems are no different. Instead of doors and windows, we examine APIs, authentication mechanisms, network boundaries, cloud infrastructure, third-party integrations, and data flows. We identify potential attack vectors, understand how an attacker might exploit them, and determine what security controls should exist before the software is deployed. The mindset is exactly the same. Only the system has changed.

One of the most common misconceptions surrounding threat modelling is that it is simply another form of security review or audit. It isn't. A security audit evaluates an implementation that already exists. Threat modelling is a collaborative activity. that should ideally be done at the design stage, and is intended to shape the implementation before security issues become expensive to fix. Because of that, threat modelling should never be performed in isolation. While the security team often facilitates the exercise, the people who know the system best are the ones who build and operate it. A successful threat modelling session should involve as many relevant stakeholders as practical: security champions, software engineers, technical architects, platform engineers, DevOps engineers, and anyone else who understands how the application actually works. The goal isn't to have security tell engineering what's wrong. It's to combine everyone's knowledge of the system to identify risks that no single person would spot alone.

Threat modelling is most effective when it's treated as part of the Secure Software Development Lifecycle (SSDLC), not as a checkpoint at the end of it. Conducting a threat model after the product has already been built often means that architectural decisions have been finalized, timelines have been committed, and implementing recommendations becomes significantly more expensive. In many cases, teams are forced to accept risks simply because fixing them would require redesigning major components.

The earlier you perform a threat model, the greater the impact it can have.

Ideally, threat modelling begins during feature design – while requirements are being discussed, diagrams are being sketched on whiteboards, and architectural decisions are still fluid. Sprint planning, design reviews, and architecture discussions are all excellent opportunities to introduce threat modelling. Of course, that ideal isn't always achievable. Teams move quickly and priorities can change. That's okay. So, at whatever stage of the SSDLC you feel "hey, we should probably do a threat model" that is a good enough stage to do a threat model at.

### The Four Questions of Threat Modelling

No matter which methodology or framework you use, nearly every threat modelling exercise revolves around four fundamental questions:

- What are we working on?
- What can go wrong?
- What are we going to do about it?
- Did we do an acceptable job?

Everything else – whether it's STRIDE, attack trees, kill chains, or any other methodology – is simply a structured way of helping you answer those four questions consistently and thoroughly.

### A Quick Refresher on Some Key Terms

Before we dive into threat modelling methodologies and processes, let's quickly revisit a few terms that you'll hear repeatedly throughout this guide – and throughout any threat modelling exercise you participate in or lead.

**Vulnerability**

A vulnerability is a weakness in a system that can potentially be exploited to compromise its security. That weakness could be anything from a software flaw and a misconfiguration to weak authentication or an outdated dependency.

It's worth mentioning an important nuance here. Not every vulnerability carries the same level of urgency. Suppose you have a service running an outdated container image that contains known vulnerabilities, but the service isn't publicly accessible and can only be reached from a tightly controlled internal network. The vulnerability still exists, but the likelihood of it being exploited is significantly lower than if the service were exposed to the internet. That doesn't mean you should ignore it – it should still be remediated – but it does mean you can make an informed decision about its priority. Risk is always a combination of multiple factors, not just the existence of a vulnerability.

**Threat**

A threat is anything capable of exploiting a vulnerability.

When people discuss attackers during threat modelling, they're usually referring to threat actors – the people or systems attempting to compromise your application. Threat actors may be external, such as cybercriminals, or internal, such as malicious insiders or compromised privileged accounts.

Ultimately, the goal of a threat actor is to compromise one or more aspects of the CIA Triad:

Confidentiality – accessing information they shouldn't have access to.
Integrity – modifying data or systems without authorization.
Availability – preventing legitimate users from accessing the service.

For example, imagine a publicly accessible website that lacks appropriate controls, allowing an attacker to deface its contents. In that case, the primary security property being compromised is integrity.

**Attack Surface vs. Attack Vector**

These two terms are often used interchangeably, but they describe different concepts. 

An application's attack surface is the collection of every possible point where an attacker might interact with or attempt to compromise the system. For a database, the attack surface might include weak authentication or authorization controls, outdated database software, excessive privileges, etc. Collectively, these represent the opportunities available to an attacker.

An attack vector, on the other hand, is the specific path an attacker uses during an attack. Suppose a database administrator uses a shared password that an attacker obtains through social engineering. The attacker then uses those credentials to access the database. The weak credential management and social engineering pathway together form the attack vector. It's one specific route through the broader attack surface.

**Impact, Probability, and Risk**

Threat modelling is ultimately about managing risk, and risk depends on two key factors:

- Impact – How severe would the consequences be if the event occurred?
- Probability (or likelihood) – How likely is the event to occur?

A simple way to understand this relationship is with an admittedly dramatic example. If a meteor were to strike Earth right now, the impact would obviously be catastrophic. Fortunately, the probability of that happening is extremely low. As a result, most organizations don't spend time preparing for meteor strikes. On the other hand, a minor application error that happens every day may have a very high probability but such a low impact that it still represents relatively little risk.

Effective risk management isn't about focusing solely on high-impact events or solely on highly probable events. It's about balancing both factors to determine where effort is best spent.

**Flaws vs. Bugs**

A good threat model should identify more flaws than bugs.

A bug is an implementation defect – a coding mistake, configuration error, or deployment issue that can often be fixed with a targeted change.

A flaw, however, is a weakness in the design or architecture of the system itself. It may involve missing trust boundaries, insecure authentication flows, poor authorization models, or assumptions that simply don't hold up against an attacker.

You want to focus more on identifying the shortcomings in the design of the system than the shortcomings in the development/deployment of the system.

Finding bugs during a threat modelling exercise is certainly valuable. Bugs often become entry points for attackers and should be addressed. But the real value of threat modelling lies in uncovering architectural flaws before they're built into the product. Design flaws are generally more expensive to fix once a system is in production, which is precisely why threat modelling aims to identify them as early as possible. In short, bugs are implementation problems. Flaws are design problems. A mature threat modelling practice focuses on both – but prioritizes finding the latter.

### Threat Modelling Methods and Approaches

Now that we've established what threat modelling is, let's look at some of the methods and approaches available to help us perform it. In this guide, we'll discuss three threat modelling methods – the techniques you can use to identify threats – and two broader approaches, which are essentially frameworks for structuring a threat modelling exercise.

Before we get into the specifics, though, there's one important point I'd like to make. These methods and approaches are the foundations upon which modern threat modelling is built. They're based on years of research and they provide an excellent starting point. But, they are not the holy texts of threat modelling. There isn't a rule that says you must use STRIDE, or that every threat model has to follow Story-Driven Threat Modelling exactly as it was originally proposed. If you deviate from these methodologies, nobody is going to revoke your security badge. Every team eventually develops its own way of threat modelling. You might start with STRIDE and discover that one part of it doesn't fit your development process. Or perhaps your team primarily uses Story-Driven Threat Modelling but has adapted certain aspects to better suit your architecture and sprint cadence. That's perfectly normal. Threat modelling is ultimately a collaborative thinking exercise, not a compliance exercise. That said, if you're leading a threat model for the first time, having some structure is invaluable. These methodologies provide that structure. Think of them as proven starting points rather than rigid rulebooks.

#### STRIDE

Developed by Microsoft, STRIDE is arguably the most mature and widely adopted threat modelling methodology in use today. Because of its popularity and practicality, we'll spend a little more time on it than on the other methods.

STRIDE is a mnemonic consisting of six categories of threats:

- S — Spoofing
- T — Tampering
- R — Repudiation
- I — Information Disclosure
- D — Denial of Service
- E — Elevation of Privilege

Rather than trying to think of every conceivable attack against a system, STRIDE encourages you to evaluate the application through each of these six lenses.

**Spoofing**

Spoofing refers to an attacker impersonating a legitimate user, service, or system component in order to gain unauthorized access. In other words, the attacker successfully defeats authentication. Common examples include:

- Stolen session tokens
- Compromised API keys
- Forged authentication credentials
- Service-to-service impersonation

Whenever you encounter authentication mechanisms during a threat model, ask yourself: Can someone pretend to be someone or something they are not?

**Tampering**

Tampering targets the integrity of a system. It involves unauthorized modification of data, configuration, or software components. Examples include:

- Altering data in transit
- Modifying records stored in a database
- Changing application configurations
- Replacing software binaries
- Manipulating API requests

Whenever a system stores or processes data, consider whether that data can be modified by someone who shouldn't be able to modify it.

**Repudiation**

Repudiation occurs when someone performs an action but can later deny having done so because sufficient evidence doesn't exist. This usually indicates weaknesses in logging, auditing, or accountability. For example, an attacker may:

- Perform unauthorized transactions
- Delete or modify audit logs
- Exploit insufficient logging to avoid attribution

A useful question to ask is: If this action occurred, could we prove who performed it?

**Information Disclosure**

Information disclosure threats compromise the confidentiality component of the CIA Triad. These threats involve sensitive information becoming accessible to unauthorized parties. Examples include:

- Data breaches
- Leaking secrets or credentials
- Exposed API responses
- Unencrypted communications
- Sensitive information appearing in logs

Whenever data is stored, transmitted, or displayed, consider whether someone could access information they shouldn't.

**Denial of Service**

Denial of Service (DoS) attacks target a system's availability. Rather than stealing information, the objective is to make the application slow, unreliable, or completely unavailable. Common examples include:

- Flooding services with excessive requests
- Resource exhaustion attacks
- Distributed Denial of Service (DDoS)
- Abuse of computationally expensive endpoints
 
Ask yourself: Can someone intentionally prevent legitimate users from using this system?

**Elevation of Privilege**

Elevation of Privilege occurs when an attacker gains permissions beyond those originally granted. This often results from weaknesses in authorization rather than authentication.

Examples include:

- A regular user becoming an administrator
- Exploiting insecure access control
- Bypassing role-based authorization
- Abusing privilege escalation vulnerabilities

The guiding question here is: Can someone do something they shouldn't be authorized to do?

| Threat                 | Property Violated | Example                                                                                                     |
| :--------------------- | :---------------- | :---------------------------------------------------------------------------------------------------------- |
| Spoofing               | Authentication    | An attacker steals the authentication token of a legitimate user and uses it to impersonate the user.       |
| Tampering              | Integrity         | An attacker abuses the application to perform unintended updates to a database.                             |
| Repudiation            | Non-repudiation   | An attacker manipulates logs to cover their actions.                                                        |
| Information Disclosure | Confidentiality   | An attacker extract data from a database containing user account info.                                      |
| Denial of Service      | Availability      | An attacker locks a legitimate user out of their account by performing many failed authentication attempts. |
| Elevation of Privilege | Authorization     | An attacker tampers with a JWT to change their role.                                                        |


At this point you might be wondering:

_"This all makes sense – but how do I actually use STRIDE during a threat modelling exercise?"_

One approach I've found useful is to use STRIDE as a mental checklist while reviewing an architecture diagram.

Start by looking for Elevation of Privilege opportunities. Ask yourself whether there's any way an attacker could bypass authorization or gain permissions beyond what they were intended to have. Authorization issues often have severe consequences, so they're a good place to begin.

Next, look for Spoofing opportunities. Examine authentication boundaries, service identities, API keys, tokens, certificates, and any mechanism that establishes identity. Could someone impersonate a legitimate user or service?

Then move on to Tampering. Focus on places where data is created, modified, or stored. Databases, object storage, message queues, APIs, and configuration repositories are all good candidates.

Once you've considered spoofing and tampering, think about Repudiation. If an attacker successfully authenticates or modifies data, would there be sufficient logs and audit trails to determine exactly what happened? Repudiation threats often emerge in the same areas where spoofing and tampering are possible.

Next, examine Information Disclosure. Follow sensitive data throughout the system. Where is it stored? Where does it travel? Could it leak through APIs, logs, backups, caches, or third-party integrations?

Finally, consider Denial of Service. Identify components whose failure would affect availability, and ask how an attacker might exhaust or overwhelm them.

This isn't the only way to apply STRIDE, nor is it the "correct" order. It's simply a practical workflow that helps ensure each category is considered without feeling overwhelmed. Like everything else in threat modelling, adapt it to suit your team and your systems.

#### PASTA

The next threat modelling method we'll look at is PASTA.

And no, not the Italian kind.

PASTA stands for Process for Attack Simulation and Threat Analysis. Compared to STRIDE, it's a relatively newer methodology and takes a noticeably different approach to threat modelling.

Where STRIDE categorizes threats into six buckets, PASTA is risk-centric. Rather than asking, "Is there spoofing here?" or "Could tampering occur?", PASTA guides you through a structured, seven-stage process that begins with understanding the business and technical objectives of the system before gradually identifying threats, analysing attack paths, and evaluating risk.

| Step                                   | Activities                                                                                                                                                                                                                                 |
| :------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. Define objectives                   | - Identify Business Objectives <br> - Identify Security and Compliance Requirements <br> - Business Impact Analysis                                                                                                                        |
| 2. Define Technical Scope              | - Capture the Boundaries of the Technical Environment <br> - Capture Infrastructure, Application, Software Dependencies                                                                                                                    |
| 3. Application Decomposition           | - Identify Use Cases, Define App. Entry Points & Trust Levels <br> - Identify Actors, Assets, Services, Roles, Data Sources <br> - Data Flow Diagramming (DFDs) / Trust Boundaries                                                         |   
| 4. Threat Analysis                     | - Probabilistic Attack Scenarios Analysis <br> - Regression Analysis on Security Events <br> - Threat Intelligence Correlation and Analytics                                                                                               |
| 5. Vulnerability & Weaknesses Analysis | - Queries of Existing Vulnerability Reports & Issues Tracking <br> - Threat to Existing Vulnerability Mapping Using Threat Trees <br> - Design Flaw Analysis Using Use and Abuse Cases <br> -Scorings (CVSS/CWSS) / Enumerations (CWE/CVE) |
| 6. Attack Modeling                     | - Attack Surface Analysis <br> - Attack Tree Development / Attack Library Mgt. <br> - Attack to Vulnerability & Exploit Analysis Using Attack Trees                                                                                        |
| 7. Risk & Impact Analysis              | - Qualify & Quantify Business Impact <br> - Countermeasure Identification and Residual Risk Analysis <br> - ID Risk Mitigation Strategies                                                                                                  |


One of PASTA's strengths is that it encourages you to think beyond individual vulnerabilities. It asks not only what could happen, but also how likely it is, what the business impact would be, and which risks deserve immediate attention.

#### DREAD

The final threat modelling method we'll discuss is DREAD.

Like STRIDE, DREAD is a mnemonic. Unlike STRIDE, however, it isn't used to discover threats – it is used to evaluate them.

Once you've identified a potential threat, DREAD provides a simple framework for estimating its severity by assigning numerical scores across five categories:

- D — Damage Potential
- R — Reproducibility
- E — Exploitability
- A — Affected Users
- D — Discoverability

| Threat           | Question                                   | Scoring                                                                                                                                                                                                                                                           |
| :--------------- | :----------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Damage Potential | How Much Damage Could the Attack Cause?    | 0: No damage <br> 5: Information disclosure <br> 8: Non-sensitive user data related to individuals or employer compromised <br> 9: Non-sensitive administrative data compromised <br> 10: Destruction of an information system; dataor application unavailability |
| Reproducibility  | How Easily Can the Attack Be Reproduced?   | 0: Difficult or impossible <br> 5: Complex <br> 7.5: Easy <br> 10: Very easy                                                                                                                                                                                      |
| Exploitability   | What’s Required to Launch the Attack?      | 2.5: Advanced programming and networking skills <br> 5: Available attack tools <br> 9: Web application proxies <br> 10: Web browser                                                                                                                               |
| Affected Users   | How Many People Would the Attack Affect?   | 0: No users <br> 2.5: Individual user <br> 6: Few users <br> 8: Administrative users <br> 10: All users                                                                                                                                                           |
| Discoverability  | How Easy Is the Vulnerability to Discover? | 0: Hard to discover the vulnerability <br> 5: HTTP requests can uncover the vulnerability <br> 8: Vulnerability found in the public domain <br> 10: Vulnerability found in web address bar or form                                                                |


The key idea behind DREAD is that it introduces a degree of consistency into risk assessment. Rather than relying solely on intuition, teams evaluate each threat against the same set of criteria, making it easier to compare risks and prioritize remediation efforts. DREAD isn't as widely used today as it once was – many organizations have adopted alternative risk-scoring methodologies – but it remains a useful way to understand the factors that contribute to the overall severity of a threat.

#### Story-Driven Threat Modelling

So far, we've looked at methods for identifying and evaluating threats. Let's now shift gears and talk about an approach to threat modelling: Story-Driven Threat Modelling. It is exactly what it sounds like: performing threat modelling against a user story or a specific piece of functionality.

Imagine you have an expense reimbursement application. Today, employees can log in, submit expense claims, and view the status of their reimbursements. Tomorrow, the product team decides to introduce a new feature that allows users to upload supporting documents for each claim. This new functionality becomes the focus of your threat model.

When performing a story-driven threat model, the first step is understanding what the system is expected to do.

Using our expense reimbursement application as an example, a user story might look something like this:

> As an employee, I want to create, upload, and view my expense claims so that my manager can review and approve them for reimbursement.

That's the intended behaviour of the system.

Once you've identified the user story, you map out how it works from a technical perspective. How does the user authenticate? Which services are involved? Where is data stored? How does it move through the system?

Data Flow Diagrams (DFDs) are particularly useful here because they allow everyone in the room to visualize the application's components, trust boundaries, and data flows. The objective is to understand the system well enough that you can begin thinking like an attacker.

Once you understand what the system is supposed to do, ask yourself a different question: How could someone intentionally misuse this functionality? This is where abuse stories come in.

An abuse story is simply the malicious counterpart to a user story.

Continuing with the same example, an abuse story might be:

> As a malicious user, I want to upload a specially crafted file that compromises the expense reimbursement application.

The user story describes intended behaviour. The abuse story describes unintended behaviour. Your role as a threat modeller is to bridge the gap between the two, which is the threat scenario. This is where you ask: How could this abuse actually happen?

For our example, several possibilities immediately come to mind:

- Uploading a malicious file that leads to remote code execution.
- Uploading content that triggers cross-site scripting (XSS).
- Exploiting insufficient validation to bypass file restrictions.
- Uploading oversized files to exhaust system resources.

Notice that we're no longer thinking in abstract terms. We're discussing concrete ways an attacker could abuse the feature being designed. That's the essence of story-driven threat modelling.

Once you've identified plausible threat scenarios, the final step is deciding what should be done about them.

Possible countermeasures for the file upload example might include:

- Strict server-side file validation.
- Restricting permitted file types.
- Malware scanning uploaded files.
- Using secure-by-default libraries for file handling.
- Performing SAST during development.
- Running DAST or penetration tests against the deployed application.
- Enforcing least-privilege permissions on storage and processing services.

The exact recommendations will depend on the architecture, the technology stack, and the risks you've identified. The important thing is that every significant threat scenario results in a discussion about appropriate mitigations.

That is Story-Driven Threat Modelling in a nutshell.

#### Mozilla's Rapid Risk Assessment

The final approach I'd like to mention isn't strictly a threat modelling methodology, but I think it's worth discussing because there are several ideas you can borrow regardless of how you conduct threat modelling.

As the name suggests, Mozilla's Rapid Risk Assessment (RRA) is a lightweight process designed to identify and prioritize risks quickly, making it particularly well suited to agile development environments where lengthy threat modelling workshops aren't always practical.

The workflow is deliberately simple. First, gather the relevant stakeholders in one place – whether that's a conference room or a video call – and establish some basic context:

- What does the application or service do?
- What data does it process?
- Where is that data stored?
- Which systems does it interact with?
- What are the important trust boundaries?

Once everyone has a shared understanding of the system, the discussion shifts to identifying potential threats and evaluating their impact.

Unlike traditional threat modelling sessions that may spend considerable time analysing individual attack scenarios, Rapid Risk Assessment emphasizes speed. The goal is to quickly identify the risks that matter most and determine whether they require immediate attention.

A useful way to frame the discussion is to ask:

- If this threat became reality, how bad would it be for the organization?
- Would it be an internal incident with limited impact?
- Would customers be affected?
- Would regulators become involved?
- Would the incident make national headlines?

Questions like these help the team focus on business impact rather than technical details alone.

Mozilla recommends keeping discussions short and focused – typically around five to ten minutes identifying a threat, followed by another five to ten minutes agreeing on possible mitigations. Everything should be documented as the session progresses, with input from all relevant stakeholders.

It's worth remembering that Rapid Risk Assessment isn't intended to replace a full threat model, a security review, or a penetration test. Instead, it's a lightweight mechanism for ensuring that important security conversations happen early and often.

Even if you never formally adopt Mozilla's methodology, there are valuable lessons to take away: involve the right people, focus on the highest risks, document decisions as you make them, and avoid turning every threat modelling session into a marathon.

### Threat Modelling For Your Org

We've now covered several threat modelling methodologies and approaches. The obvious question is: How do we actually do threat modelling at our organization?

In my experience, there is no one method that works for all, because every organization is a little different. Threat modelling should fit your engineering process, not the other way around. I tend to tweak the process depending on the engineering team I'm collaborating with and how they work. The process described below is my recommended workflow. If, over time, you discover that a particular step doesn't work well for your team, adapt it. Skip it. Replace it with something that achieves the same objective more effectively.

**Step 1: The Scoping Call**

The first meeting in any threat modelling exercise is what we refer to as the scoping call. Whenever possible, this works even better as an in-person whiteboarding session, but a virtual meeting works perfectly well too. The objectives of this meeting are threefold.

First, establish a shared understanding of what the threat modelling exercise is intended to achieve. If some participants are unfamiliar with threat modelling, spend a few minutes explaining the process and outlining what the subsequent sessions will look like. Setting expectations early helps everyone understand why they're there and how they'll contribute.

Second, gather the artefacts you'll need for the exercise. Architecture diagrams, Data Flow Diagrams (DFDs), C4 diagrams, sequence diagrams, API specifications, or any other documentation that helps explain how the application works are all valuable. Visual representations are particularly useful in remote sessions because they give everyone a common point of reference throughout the discussion. If you're conducting the exercise in person, however, don't feel constrained by documentation. A whiteboard and a room full of engineers can often produce an excellent threat model from scratch.

Finally – and perhaps most importantly – you need to define the scope of the threat model. Very few applications do only one thing. An application might support fifteen different features, integrate with dozens of services, and expose numerous APIs. Attempting to model every feature in a single workshop usually results in an exhausting and unfocused exercise. Instead, work with the product team to identify the specific features or user stories that will be covered during this threat model. There isn't a universally correct number. It simply needs to be manageable while still providing meaningful coverage of the application. Scoping also involves agreeing on system boundaries. For example, if your application consumes a third-party API, the security of that external API generally isn't within the scope of your threat model. What is in scope is how your application communicates with it, validates its responses, protects any data sent to it, and handles failures securely. Establishing these boundaries early prevents unnecessary discussions later.

**Step 2: Model One Use Case at a Time**

Once the scope has been agreed upon, schedule one or more working sessions to walk through each selected use case. The key here is to model each user story end to end. Start at the point where the user first interacts with the application – perhaps logging in or submitting a request – and follow the entire journey until the request has been processed and the response returned. As you move through the flow, continuously ask:

- What could go wrong here?
- How might an attacker abuse this step?
- What assumptions are we making?
- What security controls already exist?
- Are they sufficient?

Whenever a plausible threat is identified, discuss possible countermeasures before moving on. By the end of the session, each user story should have a documented set of identified threats and corresponding mitigation recommendations.

**Step 3: Turn Findings into Action**

A threat model only delivers value if its findings result in improvements to the product. Once the workshops have concluded, convert the identified risks into actionable engineering work. Create backlog items for the product team, document the agreed countermeasures, assign owners where appropriate, and ensure that remediation work becomes part of the normal development process rather than a standalone security initiative. Ultimately, the objective isn't simply to produce a threat model. It is to integrate threat modelling into the Secure Software Development Lifecycle (SSDLC) so that identifying and addressing security risks becomes part of how software is built.

### Revisiting the Four Questions

Earlier, we introduced the four fundamental questions of threat modelling. By now, you can probably see how they map directly to the process we've just discussed:

| Threat Modelling Question         | What It Means in Practice                                                                                                                                                          |
| :-------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What are we working on?           | Understand the application, gather the necessary documentation, and define the scope of the threat model.                                                                          | 
| What can go wrong?                | Walk through each user story and identify potential threats.                                                                                                                       |
| What are we going to do about it? | Define the security controls and countermeasures required to mitigate those threats.                                                                                               |
| Did we do an acceptable job?      | Verify that the agreed mitigations have been implemented and are effective. This validation may involve design reviews, testing, or independent verification by the security team. |


Everything we've discussed throughout this guide ultimately comes back to these four questions. Whether you use STRIDE, PASTA, Story-Driven Threat Modelling, Mozilla's Rapid Risk Assessment, or a process of your own, every successful threat model seeks to answer them thoroughly and honestly.

### Putting It into Practice: A Simple Threat Modelling Exercise

We've covered the theory, the terminology, the methodologies, and the process. Now let's put everything together with a simple example. The objective of this exercise isn't to teach another methodology or demonstrate the "correct" way to conduct a threat model. Instead, it's to show you how to think during a threat modelling exercise. Throughout this guide, we've introduced concepts such as user stories, abuse stories, threat scenarios, and countermeasures. This example simply demonstrates how they fit together in practice. You may eventually develop a different workflow that better suits your team, and that's perfectly fine. The goal here is simply to provide enough structure that you're never left wondering "okay... where do I even start?"

![Alt text](/images/threat-model/2026-07-06-threat-modeling-101.png "demo arch")

For this demonstration, imagine a simple application consisting of:

- A user interacting with a React frontend.
- A backend service exposing APIs.
- A database storing application data.

**Step 1: Understand the System**

The exact architecture isn't particularly important. What's important is understanding how the pieces communicate with one another.

The very first thing you should identify is the system's trust boundaries. A trust boundary represents a point where the level of trust changes. In our example, the end user exists outside the application's trust boundary. Every request originating from the user should therefore be treated as untrusted until proven otherwise. Similarly, imagine the application is hosted in Google Cloud but the database resides in AWS or perhaps in an on-prem data centre as part of a legacy system. Crossing into that external environment introduces another trust boundary. These boundaries deserve special attention because they frequently become the source of interesting threats. Data crossing trust boundaries often requires additional authentication, authorization, validation, encryption, or monitoring. That doesn't mean you ignore the rest of the architecture. It simply means trust boundaries are usually where you'll uncover the most valuable security discussions.

**Step 2: Do Your Homework**

Before the actual threat modelling workshop begins, spend a little time familiarizing yourself with the application. Review the architecture diagrams. Understand the authentication flow. Identify external dependencies. Run through a quick STRIDE assessment in your head. Ask yourself questions like: Where could privilege escalation occur? How are identities established? How are credentials managed? Where does sensitive data cross trust boundaries?

This isn't the threat model itself. Think of it as preparation. Spending a few minutes understanding the application beforehand often saves hours during the workshop itself.

**Step 3: Run Through The Use Cases**

Example 1: Viewing Team Snapshots

Let's start with a simple user story. 

> User Story: A user logs in and retrieves a list of virtual machine snapshots belonging to their team.

So far, everything is working exactly as intended. Now ask yourself the second question of threat modelling: What could go wrong?

An obvious abuse story is:

> Abuser Story: As a malicious user, I want to retrieve snapshots belonging to another team.

Now we've identified an attacker objective. The next step is to understand the threat scenario. How could this abuse actually happen? Perhaps the API fails to validate ownership correctly. Perhaps authorization checks are missing. Perhaps a user can simply modify a team identifier in an API request and access another team's data.

Now we discuss countermeasures.

During the workshop, the engineering team explains that:

- Every request is authenticated using OAuth.
- Authorization is enforced by a centralized API management layer.
- Users receive only the permissions required for their role, following the Principle of Least Privilege.

Excellent.

In this case, the identified threat already has appropriate mitigations. This is an important point that often gets overlooked. Finding a threat isn't evidence that the system is insecure. Sometimes the exercise simply confirms that the correct security controls already exist. That's still a successful threat model.

Example 2: Creating Virtual Machine Images

Let's look at another user story.

> User Story: A user creates a virtual machine image and stores a snapshot of it.

Again, ask yourself: What could go wrong?

An abuse story might be:

> Abuser Story: As a malicious user, I want to create a virtual machine containing unauthorized software such as cryptocurrency mining tools.

Now consider the threat scenario. Perhaps users are allowed to install arbitrary software onto their virtual machines without restriction. Perhaps there's no validation of the images being uploaded.

Possible countermeasures include:

- Allowing only approved base images.
- Restricting software installation to trusted repositories.
- Sandboxing virtual machine environments.
- Scanning images before deployment.
- Monitoring workloads for suspicious behaviour.

Again, we've followed exactly the same process: User Story → Abuse Story → Threat Scenario → Countermeasures

Example 3: Deleting Snapshots

One final example.

> User Story: A user creates and stores virtual machine snapshots.

Now consider another abuse story.

> Abuser Story: As a malicious user, I want to delete existing snapshots.

How could that happen? Perhaps ordinary users have excessive database permissions. Perhaps privileged administrative accounts aren't adequately protected. Perhaps a disgruntled database administrator deliberately deletes production snapshots.

Notice that not every threat comes from external attackers. Insider threats are equally valid considerations during threat modelling.

Possible countermeasures might include:

- Restricting delete operations to a small number of privileged administrators.
- Enforcing Role-Based Access Control (RBAC).
- Applying the Principle of Least Privilege.
- Requiring additional approval for destructive operations.
- Maintaining comprehensive audit logs of administrative actions.
- Regularly reviewing privileged access assignments.

**Conclusion: The Pattern Never Changes**

By now, you've probably noticed that every example follows the same thought process.

- Define the user story.
- Think like an attacker and create one or more abuse stories.
- Determine how each abuse story could realistically occur by developing threat scenarios.
- Agree on appropriate countermeasures.

That's really all threat modelling is.

The methodologies we've discussed – STRIDE, PASTA, Story-Driven Threat Modelling, Mozilla's Rapid Risk Assessment – are simply different ways of helping you ask the right questions. At its core, threat modelling always comes back to the same four questions we started with:

- What are we working on?
- What can go wrong?
- What are we going to do about it?
- Did we do an acceptable job?

If you consistently answer those four questions, involve the right stakeholders, and integrate the exercise into your development process, you'll be well on your way to building more secure software.
