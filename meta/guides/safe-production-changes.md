---
tags:
  - guide
  - meta/practice
  - topic/change-management
  - topic/incident
aliases: [prod-change-discipline, rollback-first, change-blast-radius]
---

# Safe production changes — discipline

Personal notes on how to make a risky change to live infrastructure without turning it into an incident. Learned the hard way.

## 1. Prepare AND verify the rollback BEFORE the forward change

Never run a forward mutation on prod until the undo path is:
- written (a script, not "I'll figure it out"),
- has the IDs/state it needs captured beforehand,
- copy-paste ready to fire,
- and ideally rehearsed on a sandbox.

The prepared rollback is what makes a risky change acceptable. When the change surprises you (it will), the revert must be one command, not improvised under pressure. A rehearsed swap+rollback on a throwaway environment first also tells you the rollback actually works.

## 2. Scripts that mutate prod must be self-documenting and leave a readable audit trail

During an incident you (or a colleague watching your screen) must be able to read, top-to-bottom, EXACTLY what was done. Not a wall of `curl`/JSON noise.

A prod script should:
- have a header comment: what it does, why, and how to roll back,
- echo each action with a timestamp + the target and before/after (not fire commands silently),
- save the mutated resource's prior state somewhere (e.g. dump the deleted object to a file) for audit and rollback,
- avoid cryptic dense one-liners on screen.

If someone screen-shares the terminal mid-incident, the output should tell the story. "Illegible during an incident" is a real defect, not a style nitpick.

## 3. Compute the blast radius against the LIVE system, not just the IaC

The most dangerous surprises come from dependencies the code does not model. IaC drift is real: dashboard-managed records, aliases, or CNAMEs can point at / flatten onto the very resource you are changing, and your `terraform plan` will not show them.

Before changing a shared/foundational resource (an apex/root DNS record, an origin, a shared load balancer, a base image):
- enumerate what points at it in the LIVE system, not only in the repo,
- specifically look for indirect references: aliases, CNAMEs flattened onto it, records with no route/rule of their own that fall through to its origin,
- assume "everything I didn't map" is bigger than "the one path I'm trying to fix".

Corollary: a change that looks surgical in the IaC ("only these 4 paths change") can have a wide real blast radius. Verify against reality before committing, and keep the rollback (rule 1) ready for the part you didn't foresee.

## TL;DR
Rollback first, ready and rehearsed. Scripts readable enough to survive a screen-share during an incident. Blast radius measured against the live system, because the code doesn't model the drift.
