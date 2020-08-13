---
name: Creating a LodeStar Release
about: A template to use for releasing
title: v1.0.0
labels: ''
assignees: ''

---

This repository contains GitHub workflows for managing releases automatically.

**Please name this issue what you would like the new release to be called. E.x. "v1.0.0"**

The following slash commands are available to use in comments on this issue:

| Command | Description | Parameters |
|---|---|---|
| `/release` | Creates a new released based on the current state of the `master` branch | `frontend=<version>`<br/>`backend=<version>`<br/>`gitapi=<version>`<br/>`dispatcher=<version>`<br/>`agnosticv=<version>`<br/>`anarchy=<version>`<br/>`poolboy=<version>` |
| `/promote` | Promotes the created release to the next environment | N/A |
| `/cancel` | Cancels the current release (and closes this issue) | N/A |
| `/finish` | Completes the current release (and closes this issue) | N/A |