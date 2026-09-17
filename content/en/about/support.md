---
weight: 3
title: "Commercial Support"
---

coost remains open source. Commercial support is available for teams using coost in production or needing in-depth support such as cross-platform adaptation, coroutine hooks, performance optimization, and team training, helping reduce risk and save time.

If you are not sure whether paid support is needed, you can request a free initial diagnosis first.

## Service Philosophy

- coost itself remains open source. Commercial support does not affect the open-source license.
- What you pay for is time, certainty, and risk reduction, not the code itself.
- Diagnose first, then quote. Requirements and boundaries are clarified as early as possible.
- Remote support is the default. On-site support is negotiated separately.
- The prices below are reference ranges. The final quote depends on assessment.

## Scope of Services

- Custom development: architecture adaptation (RISC-V / MIPS / Windows ARM64, etc.), coroutine hooks, performance optimization, custom features
- coost usage training, technical consulting, and technical training
- Annual technical support with priority response
- Troubleshooting and resolution of complex technical issues

## Service Packages

| ID | Package | Use Case | Typical Duration | Reference Price |
|---|---|---|---|---|
| CS-00 | Free Diagnosis | Identify scope before committing | 30 min | Free |
| CS-01 | Technical Consulting | Architecture, debugging, code review | Per hour | 800–1,500 CNY/hour, 1 hour minimum |
| CS-02 | Custom Development | Architecture adaptation, coroutine hooks, performance optimization, custom features | Per project | From 10k–50k CNY; complex projects at 3,000–6,000 CNY/man-day |
| CS-03 | Team Training | Get a team up to speed with coost | 1–3 days | 8k–30k CNY/day |
| CS-04 | Annual Support | Long-term production use | 1 year | 30k–100k CNY/year |
| CS-05 | Emergency Rescue | Production incidents, release blockers | Per case | From 5,000 CNY/case |

## Package Details

### CS-00 Free Diagnosis

**Who It's For**: Teams already using or planning to use coost but unsure whether paid support is needed.

**Deliverables**:

- 30-minute online meeting
- Quick assessment of the issue type: adaptation, hooks, performance, architecture, usage
- Initial suggestions and possible options
- Follow-up package quote ranges if needed

**What the Client Provides**:

- Project background, target platform, coost version
- Issue description, logs, minimal reproduction (if possible)
- Expected timeline

**Not Included**:

- Code changes
- In-depth investigation
- Written report
- Long-term Q&A

### CS-01 Technical Consulting

**Who It's For**:

- Architecture selection stage
- Specific compilation, linking, coroutine scheduling, or hook issues
- Code review or design review
- Guidance for a team to complete work independently

**Deliverables**:

- Online meeting or email/Issue consulting
- Problem analysis and proposed solutions
- Key code snippets or pseudocode
- Meeting minutes or conclusion summary
- Optional: PR review, design document review

**Reference Price**:

- 800–1,500 CNY/hour, **1 hour minimum**
- Single session: 1,500 CNY (1-hour meeting + 30-minute preparation + brief summary)
- 5-hour package: 10% off
- 10-hour package: 20% off
- 20-hour package: 25% off

**Not Included**:

- Full feature development
- Production deployment
- 24×7 response
- Fixing bugs in third-party libraries themselves

### CS-02 Custom Development

**Who It's For**:

- Need adaptation to RISC-V, MIPS, Windows ARM64, etc.
- Need to use third-party network libraries directly in coroutines (coroutine hooks)
- Need performance optimization: latency, throughput, or P99 not meeting targets
- Need custom coost features, private protocols, scheduling policies, monitoring interfaces, or toolchain integration

**Sub-categories and Reference Prices**:

| Sub-category | Content | Reference Price | Duration |
|---|---|---|---|
| Architecture Adaptation | Compilation, running, basic tests on RISC-V / MIPS, etc. | 10k–30k CNY | 1–4 weeks |
| Coroutine Hook Integration | Enable a specified third-party network library in coroutines | 20k–50k CNY | 3–6 weeks |
| Performance Optimization | Baseline testing, bottleneck analysis, tuning, re-testing | 10k–40k CNY | 1–4 weeks |
| Custom Features | Custom feature development for coost | 3,000–6,000 CNY/man-day, from 20k CNY | Per requirements |

**Deliverables**:

- Requirement confirmation and technical design
- Code patches or modules
- Test cases, examples, documentation
- Optional: upstream PR

**Acceptance Criteria**:

- Acceptance based on mutually confirmed requirements and test cases
- Architecture adaptation: compiles and runs on target platform
- Coroutine hooks: specified library connects, requests, and responds in coroutines; timeout and cancellation work as expected
- Performance optimization: re-tested in the same environment and meets agreed baseline
- Custom features: accepted per requirements document

**Not Included**:

- Third-party commercial licenses
- Hardware, cloud resources, travel
- Large-scale refactoring of client business code
- Long-term free maintenance
- Scope creep

### CS-03 Team Training

**Who It's For**:

- New teams adopting coost
- Upgrading from v3 to v4.0.0
- Internal technical sharing
- Training on coroutines, network programming, or performance tuning

**Available Formats**:

| Format | Content | Reference Price |
|---|---|---|
| Online Training | 2–4 hours per session | 8k–15k CNY/day |
| On-site Training | 1–3 days | 15k–30k CNY/day |
| Customized In-house Training | Course designed per requirements | Negotiable |

**Optional Training Topics**:

- coost architecture and core components
- Coroutine principles and scheduling
- Network programming and hooks
- Cross-platform compilation and adaptation
- Performance analysis and optimization
- Migration from v3 to v4.0.0

**Not Included**:

- Travel, accommodation, venue
- Long-term free Q&A after training
- Client business code development

### CS-04 Annual Support

**Who It's For**:

- Teams already using coost in production
- Need stable upgrades, security notifications, and priority response
- Prefer not to quote each issue separately

**Suggested Tiers**:

| Tier | Included | Reference Price |
|---|---|---|
| Basic | Priority email/Issue support, 24h response on business days, 10 consulting hours | 30k–50k CNY/year |
| Standard | 8h response on business days, 20 consulting hours, quarterly meetings | 50k–80k CNY/year |
| Advanced | 4h response on business days, 50 consulting hours, monthly meetings, upgrade support | 80k–100k CNY/year |

**Deliverables**:

- Priority Issue / email support
- Version upgrade advice
- Security vulnerability notifications
- Quarterly/monthly technical meetings
- A certain number of consulting hours or minor fixes
- Emergency coordination

**Not Included**:

- 24×7 support
- New feature custom development
- On-site support
- Third-party library responsibility

### CS-05 Emergency Rescue

**Who It's For**:

- Production crashes, hangs, or sudden performance drops
- Imminent release windows
- Need for rapid diagnosis and mitigation

**Deliverables**:

- Remote emergency access
- Problem diagnosis
- Temporary fix or workaround
- Brief post-mortem report

**Reference Price**:

- From 5,000 CNY/case
- After the first 2 hours, 1,500–2,500 CNY/hour
- 50%–100% surcharge for nights or holidays

**Not Included**:

- On-site support
- Hardware/network/cloud provider issues
- Long-term root-cause development

## Service Process

1. Free diagnosis
2. Clarify requirements and boundaries
3. Provide quote and timeline
4. Sign contract or confirm by email
5. Pay advance payment
6. Execute service
7. Acceptance and delivery
8. Final payment and after-sales

## Payment Terms

- Consulting/Training: 100% or 50% advance
- Custom Development: 50% advance, 50% after acceptance
- Annual Support: annual payment; quarterly payment negotiable
- Emergency Rescue: advance or within 24 hours
- Overtime billed hourly
- Travel expenses reimbursed at cost

## What the Client Provides

- Clear requirements
- Reproducible environment
- Logs, code, access permissions
- Designated contact person
- Timely feedback

## Exclusions

- Client business logic development
- Third-party library licensing and bug fixes
- Hardware, cloud resources, travel
- 24×7 unconditional support
- Unlimited requirement changes

## Intellectual Property and Confidentiality

- coost itself remains under its original open-source license.
- Client proprietary code remains with the client.
- Custom development deliverables are defined by contract.
- General improvements may be contributed upstream by mutual agreement.
- Both parties may sign an NDA.

## FAQ

### Does commercial support affect coost's open-source nature?

No. coost remains open source. Commercial support covers custom development, adaptation, optimization, training, and other in-depth services.

### Can I purchase consulting by the hour only?

Yes. Technical consulting starts at 1 hour and is billed hourly.

### Can we sign an NDA?

Yes. Specific terms are negotiable.

### Is remote support available?

Remote support is the default. On-site support is negotiable with travel and expenses.

### Can you provide invoices?

Negotiable. Taxes are charged separately based on actual rates.

### Can you guarantee a specific performance improvement?

We can agree on baseline metrics before the service and re-test in the same environment. We cannot guarantee numbers directly tied to business revenue.

### Are you responsible for third-party library issues?

We are not directly responsible for issues in third-party libraries, but we can assist with diagnosis, workarounds, or integration.

### What are the response times?

Depending on the package, Annual Support provides 24h / 8h / 4h response on business days. Emergency Rescue is negotiated per case.

## Contact

Please reach out with a brief description of your project and issue:

- GitHub Issues: https://github.com/idealvin/coost/issues
- Email: idealvin@qq.com

A free initial diagnosis is available to assess scope and feasibility.
