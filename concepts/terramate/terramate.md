---
tags:
  - concept
  - terraform
  - terramate
  - iac
  - orchestration
  - status/learning
created: 2026-07-22
difficulty: 3
time-to-master: 5h
---

# Terramate - Terraform Stack Orchestration

**Prerequisites**: [[terraform-fundamentals]]
**Related to**: [[terraform-reconcile-manual-drift]], [[terraform-ignore-changes-partial-ownership]]

---

## 🎯 TL;DR (30 seconds)

> Terramate is an **orchestration + code-generation layer on top of Terraform** (and OpenTofu).
> It organizes infra into many small **stacks**, generates shared HCL so you don't repeat it,
> orchestrates runs across stacks in the right order, and uses **change detection** to run
> only the stacks a commit actually touched.

**Analogy**: Terraform is one cook per station; Terramate is the head chef who knows the order of service and only re-fires the dishes that changed.

---

## 🤔 When to Use?

### ✅ Good for

- **Fleet of many stacks** (per app / per env) where plain `terraform` per folder is toil.
- **Run-only-what-changed CI**: change detection instead of applying everything.
- **DRY stacks**: generated providers, backends, common config.

### ❌ Bad for

- **A single small stack** → plain Terraform is enough; Terramate adds machinery you don't need.

---

## 📚 Key Concepts

### 1. Stacks

**Facts**: a stack = an isolated unit of Terraform state/config (a directory Terramate manages). You split infra into many stacks so blast radius and runs stay small.

**My understanding**:
The point of stacks is isolation: a change to one app's stack shouldn't force a plan/apply of everything else. Small stacks = small blast radius + fast, targeted runs.

### 2. Code generation

**Facts**: Terramate generates repeated HCL (backend, providers, shared locals) into each stack from a central definition, so you edit once instead of per-stack.

### 3. Orchestration + run order

**Facts**: Terramate runs `terraform` across stacks, respecting declared order/dependencies (e.g. network before app).

### 4. Change detection

**Facts**: using git, Terramate detects which stacks a commit changed and runs only those — key for fast, safe CI over a large fleet.

**My understanding**:
Change detection is what makes a big fleet manageable: CI plans/applies only the stacks I actually touched, not the whole repo, so a one-line change doesn't trigger dozens of applies.

**Mental model**: the head chef of a fleet — knows the order of service and only re-fires the dishes (stacks) that changed.

> [!warning] State needs a remote backend
> Terraform state is the only record of what's actually live. Keeping it **local = no backup**: lose the file and you no longer know what exists on your infra (and can't safely plan/apply). Use a remote backend (S3 + lock, TF Cloud, ...) — even more so with Terramate, where many stacks = many state files to protect.

---

## 💻 Minimal Example

### Context

A fleet of apps, one stack each; CI runs `terraform` only on the stacks a commit changed.

### Layout + run

```text
stacks/
├── app-a/   (terramate.tm.hcl + generated backend.tf/provider.tf + main.tf)
├── app-b/
└── app-c/
```
```bash
# run terraform only on stacks changed since the base branch
terramate run --changed -- terraform plan
```

A one-line change in `app-b/` plans `app-b` only, not the whole fleet.

---

## ⚠️ Pitfalls Experienced

- **Editing generated files by hand**: they get overwritten on the next generate; change the source template, not the output.
- **Assuming a change is isolated**: a shared/generated definition can fan out to every stack, so a "small" edit can mark the whole fleet as changed.

---

## 🧠 Retrieval Practice

<details>
<summary><strong>Q1:</strong> What does change detection buy you?</summary>

**Answer**: CI plans/applies only the stacks a commit touched, not the whole fleet — fast, small blast radius.
</details>

<details>
<summary><strong>Q2:</strong> Why is local state dangerous, especially with many stacks?</summary>

**Answer**: Local state = no backup. Lose it and you no longer know what's live. Many stacks = many state files → remote backend.
</details>

---

## 📊 Stats

```yaml
Total time: ~5h (learning) + hands-on during the migration
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: Terramate official docs (terramate.io); [[terraform-fundamentals]].

---

**Last update**: 2026-07-22
**Next review**: 2026-08-22
