# 🔒 Security Check Workflow for NPM Projects

## Why This Exists (The Critical Incident)

Not too long ago, the **JavaScript ecosystem was shaken** by a major supply chain attack.
The NPM account of a very well-known maintainer (author of **chalk, strip-ansi, color-convert**) was compromised.

Here’s what happened 👇

* 🚨 **Malicious versions** of packages were published to NPM (over **1 billion downloads per week** were affected).
* 🕵️ The malware wasn’t simple — it acted like a **crypto-clipper**:

  * It scanned for **crypto wallet addresses** in responses.
  * Silently swapped them with **fake addresses** that looked visually similar (using Levenshtein distance).
* 💸 It even hijacked **MetaMask transactions** by patching `eth_sendTransaction` before signing.

Developers didn’t notice at first. It only came to light because a **CI/CD pipeline broke unexpectedly**.

That’s when it hit me:
➡️ If such a massive and popular project could get compromised, **any of us could become victims overnight**.

---

## What This GitHub Action Does

To protect against this kind of attack, this workflow runs a **security audit** on your project’s dependencies — automatically, on a schedule, and on every change.

Here’s how it works:

1. **Triggers**

   * Runs whenever you push to the `main` branch.
   * Runs on every pull request.
   * Runs **daily at 6 AM UTC** (cron job) — even if no code changes happen.

2. **Steps in the Workflow**

   * ✅ Checks out your code.
   * ✅ Sets up Node.js (v20).
   * ✅ Installs dependencies using `npm ci --ignore-scripts` (ignores potentially malicious install scripts).
   * ✅ Runs `npm audit` with `--audit-level=high --omit=dev` to scan for vulnerabilities in your **production dependencies**.

3. **Why It Matters**

   * 💡 Finds known vulnerabilities in your direct & indirect dependencies.
   * 💡 Prevents accidental introduction of compromised packages.
   * 💡 Helps secure your **supply chain** — the weakest point in modern JavaScript development.
   * 💡 Acts as an **early warning system** for you and your team.

---

## Example Workflow File

```yaml
name: Security Check

on:
  push:
    branches: [ main ]
  pull_request:
  schedule:
    - cron: "0 6 * * *" # daily

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci --ignore-scripts
      - run: npm audit --audit-level=high --omit=dev
```

---

## Why You Should Care

Software supply chain attacks are **not theory anymore**. They are happening right now:

* 🧨 Packages you trust can be compromised.
* 🧨 Malicious code can sneak in through **indirect dependencies** you didn’t even install yourself.
* 🧨 CI/CD pipelines, wallets, and production servers can all be affected.

This Action is a **small but powerful step** toward protecting your project and your users.
It makes sure you **never ignore security warnings** and that you catch issues *before* they ship.

---

## How to Enable It in Your Repo

1. Create the folder (if not already there):

   ```
   mkdir -p .github/workflows
   ```

2. Add the workflow file:

   ```
   .github/workflows/security-check.yml
   ```

3. Copy the code above.

4. Commit & push:

   ```bash
   git add .github/workflows/security-check.yml
   git commit -m "Add security audit workflow"
   git push origin main
   ```

---

## Show It Off: Status Badge

You can display the workflow status in your repo’s README by adding this badge:

```markdown
![Security Check](https://github.com/your-username/your-repo/actions/workflows/security-check.yml/badge.svg)
```

It will show ✅ green when your repo is safe, ❌ red when an issue is found.

---

## Final Words

This workflow isn’t a silver bullet — but it’s a **solid line of defense** against the next supply chain attack.

The recent NPM incident proved that **we can’t just trust our dependencies blindly**.
We need to continuously **audit, monitor, and react quickly**.

If you find this useful, ⭐ star this repo and share it with your team.
The more developers adopt practices like this, the harder it becomes for attackers to succeed.

Stay safe, keep building 🚀
