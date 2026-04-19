---
date: '2026-04-13T08:58:43Z'
draft: true
title: 'Linux'
image: '/images/linux.webp'
tags: ["betriebssysteme","linux"]
---


{{< mermaid >}}
---
config:
    theme: "base"
    gitGraph:
        showCommitLabel: false
        showBranches: true
        mainBranchName: 'Debian'
        parallelCommits: true
---
gitGraph TB:
    commit
    commit
    branch Ubuntu
    commit
    checkout Debian
    commit
    commit
    commit
    checkout Ubuntu
    branch "Lubuntu"
    commit
    commit
    checkout Ubuntu
    branch "Xubuntu"
    commit
    commit
    branch "Kubuntu"
    checkout Ubuntu
    branch "Linux Mint"
    commit
    commit
    checkout Ubuntu
    branch "Pop!_OS"
    commit
    commit
    checkout Ubuntu
    commit
    commit
{{< /mermaid >}}