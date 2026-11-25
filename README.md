# Kubernetes Administration: Practical Study Companion

This is the learner-materials repository for Emmanuel Corels Academy's
**CKA Preparation: Kubernetes Administration Scenario Tests** course.

The practice questions help you decide what to investigate. This companion
helps you follow those decisions with practical work in your own disposable
cluster. It is independent preparation, not an official Kubernetes or Linux
Foundation examination environment.

## Start here

1. Read [the practical study guide](CKA-Practical-Study-Guide.md), including the
   safe-workspace instructions, before running any commands.
2. Open [the printable HTML guide](index.html) after downloading the repository,
   or use the Markdown guide directly on GitHub.
3. Inspect [the optional lab manifest](scenario-lab.yaml). Choose a new namespace
   that you own and adapt the manifest before applying it to your lab.
4. Record your own observations in [the evidence notebook](evidence-notebook.tsv).
   An expected result in a question is not evidence that your lab produced it.

The guide includes official Kubernetes documentation links for each investigation.
Check the current Linux Foundation exam instructions and software version before
booking an exam. These materials do not reproduce the performance-based exam.

## Download the files

Use **Code → Download ZIP** on GitHub, or download the versioned learner bundle
from this repository's Releases page. No Studio login or paid lab subscription is
required to download the materials.

## Safety and scope

- Use a disposable cluster you own, never production or an exam environment.
- Inspect context, cluster endpoint and namespace before making changes.
- Do not publish kubeconfig files, access tokens, private keys or customer data.
- Review namespace and object ownership before cleanup; do not use broad deletes.
- Container images and registry access may require adaptation in your environment.
- The example manifests are study starting points, not production configurations.

Keep your own evidence of commands, events and outcomes. The repository does not
claim that every exercise has been executed in your environment or guarantee a
certification result.
