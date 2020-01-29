---
title: "Running a CMSSW job"
teaching: 10
exercises: 5
questions:
- "How can I run CMSSW in GitLab CI?"
- "How can avoid compiling my code for each job?"
objectives:
- "Successfully run a test job of a simplified Z to leptons analysis"
- "Use GitLab artifacts to pass compiled analysis code"
keypoints:
- "The use of artifacts allows passing results of one step to the other"
- "Since artifacts are write-protected, the directory needs to be copied before running CMSSW"
---
FIXME

[gitlab-artifacts][GitLab documentation for using artifacts]

{% include links.md %}

