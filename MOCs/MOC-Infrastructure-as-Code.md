# MOC - Infrastructure as Code

> **What**: Learning path for IaC tools - provisioning and configuration management.
>
> **Who**: DevOps engineers automating infrastructure deployment.
>
> **Why**: Deploy reproducible, version-controlled infrastructure across any cloud.

---

## 🎯 Learning Objective

Master Infrastructure as Code to provision cloud resources (Terraform) and configure servers (Ansible) in an automated, reproducible way. Replace manual setup with declarative code.

**Real-world use case**: Deploy identical dev/staging/prod environments with a single command, track changes in Git.

---

## 📚 Learning Path

### Phase 1: IaC Fundamentals (27h total) ✅ Completed

**Goal**: Understand the core workflow of Terraform and Ansible.

1. **[[concepts/terraform/terraform-fundamentals]]** ⭐⭐⭐⭐ (15h)
   - **Why first**: Terraform provisions the infrastructure (servers, networks, storage)
   - **You'll learn**: init/plan/apply workflow, tfstate, providers, basic resources

2. **[[concepts/ansible/ansible-fundamentals]]** ⭐⭐⭐ (12h)
   - **Why now**: Ansible configures what Terraform creates
   - **You'll learn**: Playbooks, inventory, idempotence, modules, handlers

---

### Phase 2: Advanced Patterns (planned)

**Goal**: Production-ready IaC with modules, remote state, and secrets.

3. **[[concepts/terraform/terraform-modules]]** ⭐⭐⭐ (planned)
   - **Why now**: Reusable, shareable infrastructure code
   - **You'll learn**: Module structure, inputs/outputs, registry modules

4. **[[concepts/terraform/terraform-remote-state]]** ⭐⭐⭐ (planned)
   - **Why now**: Team collaboration requires shared state
   - **You'll learn**: S3 backend, state locking with DynamoDB

5. **[[concepts/ansible/ansible-vault]]** ⭐⭐ (planned)
   - **Why now**: Secure secrets management in playbooks
   - **You'll learn**: Encrypting variables, vault passwords, best practices

6. **[[concepts/ansible/ansible-roles]]** ⭐⭐⭐ (planned)
   - **Why now**: Organize complex playbooks
   - **You'll learn**: Role structure, Galaxy, reusable roles

---

### Phase 3: CI/CD Integration (planned)

**Goal**: Automate IaC in pipelines.

7. **[[concepts/terraform/terraform-cicd]]** ⭐⭐⭐⭐ (planned)
   - **Why now**: GitOps workflow for infrastructure
   - **You'll learn**: Plan in PR, apply on merge, Atlantis/Spacelift

---

## 🛠️ Practice Project

**Apply your knowledge**: [[projects/2025-01-aws-terraform-ansible/learnings]]

**What you'll build**: WordPress on AWS EC2 - Terraform provisions, Ansible configures.

**Why this project**: Real combination of both tools with Docker deployment.

---

## 📊 Progress Tracking

```yaml
Total estimated time: 27h completed / ~50h planned
Difficulty: ⭐⭐⭐⭐ (4/5)
Prerequisites: [[MOCs/MOC-Linux-Security]], [[MOCs/MOC-Cloud-AWS]]
```

| Phase | Status | Time |
|-------|--------|------|
| Phase 1: IaC Fundamentals | ✅ Completed | 27h |
| Phase 2: Advanced Patterns | ⏳ Planned | ~20h |
| Phase 3: CI/CD Integration | ⏳ Planned | ~10h |

---

## 🔄 Next Steps

After completing this path, you'll be ready for:
- **[[MOCs/MOC-Kubernetes]]**: Deploy K8s with Terraform, configure with Helm
- **GitOps**: ArgoCD, Flux for continuous delivery

---

## 📖 Quick Reference

**Core concepts to remember**:
1. **Terraform = Provisioning**: Creates infrastructure (servers, networks, storage)
2. **Ansible = Configuration**: Configures what Terraform creates (packages, services, files)
3. **Idempotence**: Run N times = same result. Essential for both tools.
4. **tfstate = Source of truth**: Without it, Terraform doesn't know what exists

**Common pitfalls**:
- EC2 created but SSH not ready (add wait loop before Ansible)
- Different environments between test and production (use identical images)
- Forgetting egress rules (containers can't reach Internet)
- Committing tfstate with secrets (use remote backend)

**Cheatsheets**: [[cheatsheets/terraform/terraform-commands]] | [[cheatsheets/ansible/ansible-commands]]

---

**Created**: 2025-01-22
**Last updated**: 2025-01-22
