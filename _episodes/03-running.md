---
title: "Running a CMSSW job"
teaching: 10
exercises: 10
questions:
- "How can I run CMSSW in GitLab CI?"
- "How can avoid compiling my code for each job?"
objectives:
- "Successfully run a test job of a simplified Z to leptons analysis"
- "Use GitLab artifacts to pass compiled analysis code"
keypoints:
- "A special CMSSW image is required to successfully run CMSSW jobs"
- "Running on CMS data requires a grid proxy"
- "The use of artifacts allows passing results of one step to the other"
- "Since artifacts are write-protected, the directory needs to be copied before running CMSSW"
---

Being able to set up CMSSW and compiling code in GitLab, the next step is to
run test jobs to confirm that the code yields the expected results.

> ## Fair use
> Please remember that the provided runners are shared among all users, so
> please avoid massive pipelines and CI stages with more than 5 jobs in
> parallel or that run with a parallel configuration higher than 5.
>
> If you need to run these pipelines please deploy your own private runners
> to avoid affecting the rest of the users.
{: .callout}

## Requirements for running CMSSW

In most cases, you will run your tests on centrally produced files. In order
to be able to access those, you will require a grid proxy valid for the CMS
virtual organisation (VO).

We'll use a single file from the
[/DYJetsToLL_M-50_HT-100to200_TuneCP5_13TeV-madgraphMLM-pythia8/RunIIFall17MiniAODv2-PU2017_12Apr2018_94X_mc2017_realistic_v14-v1/MINIAODSIM](https://cmsweb.cern.ch/das/request?instance=prod/global&input=file+dataset%3D%2FDYJetsToLL_M-50_HT-100to200_TuneCP5_13TeV-madgraphMLM-pythia8%2FRunIIFall17MiniAODv2-PU2017_12Apr2018_94X_mc2017_realistic_v14-v1%2FMINIAODSIM) data set: `/store/mc/RunIIFall17MiniAODv2/DYJetsToLL_M-50_HT-100to200_TuneCP5_13TeV-madgraphMLM-pythia8/MINIAODSIM/PU2017_12Apr2018_94X_mc2017_realistic_v14-v1/50000/E43E4210-7742-E811-9430-AC1F6B23C96A.root`.

## Using artifacts to compile code only once

[GitLab documentation for using artifacts][gitlab-artifacts]

{% include links.md %}

