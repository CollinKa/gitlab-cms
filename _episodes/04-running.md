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

Being able to set up CMSSW and to compile code in GitLab, and knowing how
to access CMS data, the next step is to run test jobs to confirm that the
code yields the expected results.

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
virtual organisation (VO) as described in the previous section. For files
located on EOS, please check the section on
[private information/access control][lesson-gitlab-secrets]
from the
[Continuous Integration / Continuous Development (CI/CD)][lesson-gitlab]
on how to get a Kerberos token via `kinit` (we won't be using this here).

For the analysis example provided in this lessons, we'll use a single file
from the [/DYJetsToLL_M-50_HT-100to200_TuneCP5_13TeV-madgraphMLM-pythia8/RunIIFall17MiniAODv2-PU2017_12Apr2018_94X_mc2017_realistic_v14-v1/MINIAODSIM](https://cmsweb.cern.ch/das/request?instance=prod/global&input=file+dataset%3D%2FDYJetsToLL_M-50_HT-100to200_TuneCP5_13TeV-madgraphMLM-pythia8%2FRunIIFall17MiniAODv2-PU2017_12Apr2018_94X_mc2017_realistic_v14-v1%2FMINIAODSIM) data set: `/store/mc/RunIIFall17MiniAODv2/DYJetsToLL_M-50_HT-100to200_TuneCP5_13TeV-madgraphMLM-pythia8/MINIAODSIM/PU2017_12Apr2018_94X_mc2017_realistic_v14-v1/50000/E43E4210-7742-E811-9430-AC1F6B23C96A.root`.
This file is set in
[`ZPeakAnalysis/test/MyZPeak_cfg.py`](https://gitlab.cern.ch/awesome-workshop/payload-gitlab-cms/blob/master/ZPeakAnalysis/test/MyZPeak_cfg.py#L9).

## Executing `cmsRun`

In principle, all we need to do is compile the code as demonstrated in
[episode 2]({{ page.root }}{% link _episodes/02-compiling.md %}),
adding the grid proxy as just done in
[episode 3]({{ page.root }}{% link _episodes/03-vomsproxy.md %}),
and then execute the `cmsRun` command. Mind that do not need the
`git cms-addpkg PhysicsTools/PatExamples` command here anymore,
i.e. remove it in the following! Putting this together, the
additional commands to run would be:

~~~
cd ${CMSSW_BASE}/src/AnalysisCode/ZPeakAnalysis/
cmsRun test/MyZPeak_cfg.py
ls -l myZPeak.root
~~~
{: .language-bash}

where the last command just checks that an output file has been created.
However, imagine that you would like to run test jobs on more than one file
and to speed things up do this in parallel. This would mean that you would
have to compile the code *N* times, which is a waste of resources and time.
Instead, we can pass the compiled code from the compile step to the run step
as described below.

## Using artifacts to compile code only once

[Artifacts][lesson-gitlab-artifacts] have been introduced to you as part of the
[Continuous Integration / Continuous Development (CI/CD) lesson][lesson-gitlab].
You can find more detailed information in the
[GitLab documentation for using artifacts][gitlab-artifacts].

> ## Artifacts are write-protected
> One important thing to note is that artifacts are write-protected. You
> cannot write into the artifact directory in any of the following steps.
{: .callout}

For the compiled code to be available in the subsequent steps, the directories
that should be provided need to be listed explicitely. The `yaml` code from
the compilation step in
[episode 2]({{ page.root }}{% link _episodes/02-compiling.md %})
needs to be extended as follows:

~~~
artifacts:
  # artifacts:untracked ignores configuration in the repository’s .gitignore file.
  untracked: true
  expire_in: 20 minutes
  paths:
    - ${CMSSW_RELEASE}
~~~
{: .language-yaml}

As path we use `${CMSSW_RELEASE}`, i.e. the full CMSSW area. Since this area
is write protected, we need to copy the whole area to a new directory and
recursively add write permissions again. In the following, this new workarea
will have to be used:

~~~
script:
  # ...
  - mkdir run
  - cp -r ${CMSSW_RELEASE} run/
  - chmod -R +w run/${CMSSW_RELEASE}/
  - cd run/${CMSSW_RELEASE}/src
  - cmsenv
~~~
{: .language-yaml}

> ## Exercise: Run CMSSW using the artifact from the compile step
>
> You should now have all required ingredients to be able to extend the
> `.gitlab-ci.yml` file such that you can reuse the compiled code in the
> `cmsRun` step.
>
{: .challenge}

> ## Solution: Run CMSSW using the artifact from the compile step
>
> A possible implementation could look like this:
>
> ~~~
> cmssw_run:
>   image:
>     name: gitlab-registry.cern.ch/clange/cmssw-docker/cc7-cms:latest
>     entrypoint: [""]
>   tags:
>     - cvmfs
>   variables:
>     CMS_PATH: /cvmfs/cms.cern.ch
>     EOS_MGM_URL: "root://eoscms.cern.ch"
>     CMSSW_RELEASE: CMSSW_10_6_8_patch1
>   script:
>     - shopt -s expand_aliases
>     - set +u && source ${CMS_PATH}/cmsset_default.sh; set -u
>     - mkdir run
>     - cp -r ${CMSSW_RELEASE} run/
>     - chmod -R +w run/${CMSSW_RELEASE}/
>     - cd run/${CMSSW_RELEASE}/src
>     - cmsenv
>     - mkdir -p ${HOME}/.globus
>     - printf $GRID_USERCERT | base64 -d > ${HOME}/.globus/usercert.pem
>     - printf $GRID_USERKEY | base64 -d > ${HOME}/.globus/userkey.pem
>     - chmod 400 ${HOME}/.globus/userkey.pem
>     - printf ${GRID_PASSWORD} | base64 -d | voms-proxy-init --voms cms --pwstdin
>     - cd AnalysisCode/ZPeakAnalysis/
>     - cmsRun test/MyZPeak_cfg.py
>     - ls -l myZPeak.root
> ~~~
> {: .language-yaml}
{: .solution}

> ## Bonus: Store the output ROOT file as artifact
>
> It could be useful to store the output ROOT file as an artifact so that you
> simply download it after job completion. Do you know how to do it?
> Hint: you need to provide the full path to it.
>
{: .testimonial}

{% include links.md %}

