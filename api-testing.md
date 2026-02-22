# API Testing for Bug Bounty Hunters: A Comprehensive Guide

## Introduction

As a bug bounty hunter, API testing is like hunting for buried treasure in a vast digital ocean. APIs are the hidden plumbing of modern web applications, often exposing sensitive data or functionality that frontend interfaces don't reveal. Think of yourself as a digital archaeologist: you're not just poking around; you're systematically excavating layers to uncover flaws that could lead to data leaks, unauthorized access, or even full system compromise. The goal? Find vulnerabilities before malicious actors do, report them ethically, and get rewarded.

This guide dives deep into API reconnaissance and testing, structured as a methodology for planning your hunts. I'll explain concepts with real-life examples and analogies to make them stick. We'll cover identifying endpoints, supported methods, content types, hidden parameters, mass assignment vulnerabilities, and prevention tips. By the end, you'll have a solid framework to approach API bug bounties like a pro.

## Thinking Like a Bug Bounty Hunter

Bug bounty hunters aren't random hackers—they're strategic thinkers who maximize efficiency to find high-impact bugs quickly. Here's how to embody that mindset in API testing:

- **Recon is King**: APIs are often undocumented or poorly secured because developers assume they're "internal." Your job is to map the unknown. Imagine you're a spy in enemy territory: gather intel without alerting defenses (e.g., rate limits or WAFs).
  
- **Assume Nothing**: Documentation might lie or be outdated. Test everything—endpoints, methods, parameters. A "read-only" API might accidentally allow writes if not properly restricted.

- **Exploit the Overlooked**: Bugs like mass assignment thrive on developer oversights. Think like an attacker: "What if I add this extra parameter? What if I switch content types?"

- **Risk vs. Reward**: Prioritize testing low-impact areas first (e.g., non-critical endpoints) to avoid bans. Focus on payout potential—auth bypass or data exposure often yields big bounties.

- **Tooling and Automation**: Use Burp Suite, Postman, or custom scripts, but remember: tools amplify your brain, not replace it. Chain findings—e.g., use recon from one endpoint to fuzz others.

- **Ethics and Scope**: Always stay within program rules. Report findings clearly, with PoCs (Proofs of Concept) that demonstrate impact without harm.

Real-Life Example: In 2019, a hunter found a mass assignment bug in Shopify's API, allowing privilege escalation to admin. By inspecting response objects and adding hidden parameters like "role: admin," they bypassed auth. Bounty: $20,000+. Analogy: It's like finding a house key under the doormat—developers left it there accidentally, but you spotted it.

## Methodology: How to Plan Your API Bug Hunt

Planning is crucial to avoid wasting time on dead ends. Here's a step-by-step methodology, phased like a military operation. Adapt it per target, and document everything for your reports.

### Phase 1: Preparation (Intel Gathering)
- **Define Scope**: Review the bug bounty program's rules. What APIs are in-scope? (e.g., *.api.example.com). Note any restrictions like no DDoS or destructive testing.
- **Tool Setup**: Configure Burp Suite for proxying, Intruder for fuzzing, and extensions like JS Link Finder or Param Miner. Use Postman for manual testing.
- **Initial Recon**: Browse the app in a web browser while proxying traffic. Identify API calls via Burp's HTTP history or browser dev tools.
- **Time Estimate**: 1-2 hours. Goal: Build a target list of potential API base paths (e.g., /api/v1/).

Analogy: This is like scouting a battlefield—map the terrain before charging in.

### Phase 2: Discovery (Mapping the Attack Surface)
- **Find Documentation**: Crawl with Burp Scanner or manually check common paths (/api, /swagger.json, /openapi.yaml). Use Intruder with wordlists for variations.
- **Endpoint Enumeration**: Analyze JS files for references. Fuzz paths with Intruder (e.g., /api/users/%s where %s is "update," "delete"). Test OPTIONS method on known endpoints.
- **Parameter Hunting**: Use Param Miner to guess params. Inspect responses for object fields that might bind via mass assignment.
- **HTTP Methods and Content Types**: Cycle methods (GET, POST, PUT, DELETE, PATCH, OPTIONS) with Intruder. Switch Content-Type (JSON, XML, form-data) and observe behavior.
- **Rate Limiting & Auth Check**: Note any tokens, keys, or limits. Test for weak auth (e.g., missing JWT validation).
- **Time Estimate**: 2-4 hours. Goal: Compile a list of 20-50 endpoints with supported methods/params.

Real-Life Example: In Twitter's (now X) API, a hunter used endpoint fuzzing to find an undocumented /api/1.1/account/settings endpoint that leaked user data without proper auth. By switching to XML content type, they bypassed JSON-specific filters. Bounty: $5,000. Analogy: Like fishing with different baits—JSON might not bite, but XML reels in the vuln.

### Phase 3: Testing (Exploitation Attempts)
- **Basic Probes**: Send invalid data to trigger errors (e.g., disclose stack traces). Test for injection (SQLi, XSS) in params.
- **Hidden Endpoint/Param Testing**: Fuzz with app-specific wordlists. For mass assignment, add fields from GET responses to PATCH/POST requests (e.g., "isAdmin: true").
- **Edge Cases**: Test rate limits by bursting requests. Try invalid methods/content types for info leaks.
- **Chaining Vulns**: If you find a hidden param, combine with others (e.g., IDOR + mass assignment).
- **Time Estimate**: 3-6 hours. Goal: Identify 5-10 potential bugs with PoCs.

Analogy: This is the heist—use your map to crack safes, but start with the smallest to avoid alarms.

### Phase 4: Validation and Reporting (Extraction and Exit)
- **Impact Assessment**: For each finding, demonstrate harm (e.g., "This allows unauthorized admin access, leaking all user data").
- **Reproduce and Mitigate**: Test fixes if possible. Suggest preventions (e.g., allowlist params).
- **Report**: Use the program's template. Include steps, PoC code, and screenshots.
- **Time Estimate**: 1 hour per bug. Goal: Submit high-quality reports.

Real-Life Example: A Facebook API bug in 2018 allowed mass assignment of "verified" status via hidden params in profile updates. Hunter reported it, preventing fake verified accounts. Bounty: $10,000+. Analogy: Like returning lost treasure for a reward—instead of keeping it.

### Phase 5: Iteration and Learning
- Review what worked/didn't. Update wordlists/tools.
- Share anonymized findings on blogs (if allowed) to build rep.

Total Time: 7-13 hours per target. Scale based on complexity.

## In-Depth Explanation of Key Concepts

### API Recon: Uncovering the Hidden
Recon is about expanding the attack surface. Start with passive methods (browsing) to avoid detection.

- **Discovering Documentation**: Even "private" docs might leak via /swagger or app crawls. Use Burp to find them.
- **Identifying Endpoints**: Look for /api/ patterns. Extract from JS: e.g., a file might have `fetch('/api/books')`.
- **Supported HTTP Methods**: Not all endpoints advertise methods. OPTIONS can reveal them, but test manually to confirm.
- **Content Types**: APIs might parse JSON securely but choke on XML, leading to XXE vulns.

Lab Tip: In "Finding and exploiting an unused API endpoint," you might fuzz to find /api/old/v1/users, which is forgotten but vulnerable.

### Hidden Endpoints and Parameters
- **Fuzzing with Intruder**: Load a wordlist of actions (update, delete) and insert into paths.
- **Param Discovery**: Tools like Param Miner guess thousands. App-specific: If it's a banking app, try "balance," "transfer."

### Mass Assignment Vulnerabilities
This happens when frameworks auto-bind request params to objects without filtering.

- **Identification**: Spot in responses (e.g., {"id":123, "isAdmin":false}). Add to requests.
- **Exploitation**: Set "isAdmin":true in updates. Test with invalid values to confirm binding.
- **Prevention**: Use allowlists for bindable fields; blocklist sensitives like roles.

Lab Tip: In "Exploiting a mass assignment vulnerability," escalate privileges by binding hidden fields.

### Preventing Vulnerabilities (Hunter's Advice to Devs)
As a hunter, suggest these in reports:
- Secure docs behind auth.
- Keep docs current.
- Allowlist methods/content types.
- Generic errors only.
- Protect all API versions.
- For mass assignment: Explicit binding, no auto-magic.

Analogy: APIs are like restaurant kitchens—front of house (UI) is polished, but back (API) might have rats (vulns). Secure the back door!

## Real-Life Examples and Analogies Summary

- **Example: Uber API Leak (2014)**: Undocumented endpoints exposed ride data. Hunter fuzzing paths found them. Analogy: Like finding an unlocked side door in a fortress.
- **Example: Parler Data Breach (2021)**: Mass assignment allowed scraping all posts via incremental IDs. Analogy: A library where asking for book #1 to #1M gives you the whole catalog unchecked.
- **General Analogy**: API testing is chess—recon is opening moves, testing is mid-game attacks, reporting is checkmate. Play smart, not hard.

## Resources
- Tools: Burp Suite, Postman, ZAP.
- Wordlists: SecLists on GitHub.
- Learning: PortSwigger Academy, Bug Bounty Hunter Methodology (Intigriti).

Happy hunting! Report responsibly. 🚀
