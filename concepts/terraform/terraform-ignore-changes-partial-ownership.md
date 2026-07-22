---
tags:
  - concept
  - terraform
  - lifecycle
  - iac
  - status/learning
created: 2026-07-21
difficulty: 2
time-to-master: 6h
---

# ignore_changes & Partial Ownership - Terraform

**Prerequisites**: [[terraform-fundamentals]]
**Related to**: [[terraform-reconcile-manual-drift]]

---

## 🎯 TL;DR (30 seconds)

> `lifecycle { ignore_changes = [...] }` lets Terraform own a resource **partially**:
> it manages the resource's existence and most attributes, while an external system
> (deploy pipeline, another tool) owns specific attributes.

**Analogy**: shared flat: Terraform owns the apartment, the deploy pipeline repaints one room; TF must not repaint it back on every apply.

---

## 🤔 When to Use?

### ✅ Good for

- **Externally-mutated attribute**: content deployed by CI, tags managed elsewhere.
- **Killing a perpetual diff**: stop overwriting on every `apply`.

### ❌ Bad for

- **Silencing a plan blindly**: ignoring a structural attribute without knowing who writes it.

---

## 📚 Key Concepts

### 1. ignore_changes

**Facts**: list of attributes TF ignores during diff. The resource stays managed (create/destroy), only those attributes are left to the external owner.

**My understanding**:
`ignore_changes` is a **contract**: "I own the resource, you own this field". It lets TF keep managing the resource without fighting an external owner over specific attributes. The trap is using it to *hide* drift instead of to *declare* ownership.

**Mental model**: a shared flat — Terraform owns the apartment (walls, lease), the deploy pipeline repaints one room; TF must not repaint over it every apply.

### 2. Partial ownership

Who writes what must be explicit. Document the ignored attribute AND its real owner.

### 3. "Assets-only" trap

**Observed fact**: an "assets-only" partial ownership can fail if the provider still diffs a required attribute → verify the `plan` is **truly empty** after apply, not just "almost empty".

---

## 💻 Minimal Example

### Context

Terraform owns the resource; a deploy pipeline owns its content/assets.

### HCL

```hcl
resource "cloudflare_worker_script" "app" {
  name    = "my-app"
  content = file("placeholder.js")  # real content is pushed by the deploy pipeline

  lifecycle {
    ignore_changes = [content]       # don't revert the pipeline on every apply
  }
}
```

Proof it's clean: a second `apply` reports **No changes**.

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: ignoring an attribute you don't understand

**Symptom**:

```text
The plan is quiet, but the live resource drifts in ways TF no longer reports.
```

**What went wrong**: `ignore_changes` was used to silence a noisy diff without knowing who writes the attribute → blind drift.

**Solution**: ignore only what a *known* external owner writes, and document that owner. Verify the plan is truly empty after apply.

**Lesson**: `ignore_changes` declares ownership; it must not hide it.

---

## 🧠 Retrieval Practice

<details>
<summary><strong>Q1:</strong> What is `ignore_changes` for?</summary>

**Answer**: Declaring partial ownership — TF manages the resource but leaves specific attributes to an external owner. Not for hiding drift.
</details>

<details>
<summary><strong>Q2:</strong> How do you prove partial ownership is clean?</summary>

**Answer**: A second `apply` reports an empty plan (No changes).
</details>

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
