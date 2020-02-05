---
title: "Compiling a CMSSW package"
teaching: 10
exercises: 5
questions:
- "How can I compile my CMSSW package using GitLab CI?"
- "How do I add other CMSSW packages?"
objectives:
- "Successfully compile CMSSW example analysis code in GitLab CI"
keypoints:
- "For code to be compiled in CMSSW, it needs to reside within the work area's `src` directory."
- "The code needs to be copied manually using the CI script."
- "When using commands such as `git cms-addpkg`, the git configuration needs to be adjusted/set first."
---
Now that you know how to get a CMSSW environment, it is time to do something useful with it.

## Compiling code within the repository

For your analysis to be compiled with CMSSW, it needs to reside in the
workarea's `src` directory, and in there follow the directory structure of
two subdirectories (e.g. `AnalysisCode/MyAnalysis`) within which there can be
`src`, `interface`, `plugin` and further directories. Your analysis code
under version control will usually not contain the CMSSW workarea, but either
contain the analysis code at the lowest level or maybe collected in one
directory to disentangle it from your configuration files such as the `.gitlab-ci.yml` file.

## Adding CMSSW packages

Assuming that you would like to check out CMSSW packages using the commands
described in the [CMSSW FAQ][cmssw-faq], a couple of additional settings need
to be applied. For instance, try running the following command in GitLab CI
after having set up CMSSW:

~~~
git cms-addpkg PhysicsTools/PatExamples
~~~
{: .language-bash}

This will fail:

~~~
Cannot find your details in the git configuration.
Please set up your full name via:
    git config --global user.name '<your name> <your last name>'
Please set up your email via:
    git config --global user.email '<your e-mail>'
Please set up your GitHub user name via:
    git config --global user.github <your github username>
~~~
{: .output}

You will have to set the config as described above to make things work. Alternatively, you can also just create a `.gitconfig` in your repository and use it as described [here][custom-gitconfig].

> ## Relevance of settings
> Since you will probably not want to push changes to GitHub for inclusion
> from the CI, it doesn't really matter what you set. You can set some
> arbitrary values, but mind that if you use a username that doesn't exist
> on GitHub, you will see a warning (here `anonymous` was set):
>
> ~~~
> You don't seem to have a GitHub accout, or your GitHub username (anonymous) is not correct.
>  (anonymous) is not correct.
>  You can work locally, but you will not be able to push your changes to > tHub for inclusion
>  in the official CMSSW distribution.
>  You can correct your GitHub user name via:
>      git config --global user.github <your github username>
>  To create a personal repository:
>      visit to https://github.com/ and register a new account
>      visit to https://github.com/cms-sw/cmssw and click on the Fork button
>      select the option to fork the repository under your username (anononymous)
> ~~~
> {: .output}
{: .callout}

A complete `yaml` fragment that checks out a CMSSW package after having set up CMSSW and then compiles the code looks as follows:

~~~
cmssw_addpkg:
  stage: compile
  tags:
    - cvmfs
  variables:
    CMS_PATH: /cvmfs/cms.cern.ch
    CMSSW_RELEASE: CMSSW_10_6_8_patch1
    GIT_USER_NAME: 'Anonymous Nonamious'
    GIT_USER_EMAIL: anonymous@cern.ch
    GIT_USER_GITHUB: cmsbot
  script:
    - shopt -s expand_aliases
    - set +u && source ${CMS_PATH}/cmsset_default.sh; set -u
    - cmsrel ${CMSSW_RELEASE}
    - cd ${CMSSW_RELEASE}/src
    - cmsenv
    # If within CERN, we can speed up interaction with CMSSW:
    - export CMSSW_MIRROR=https://:@git.cern.ch/kerberos/CMSSW.git
    - export CMSSW_GIT_REFERENCE=/cvmfs/cms.cern.ch/cmssw.git.daily
    - git config --global user.name "${GIT_USER_NAME}"
    - git config --global user.email "${GIT_USER_EMAIL}"
    - git config --global user.github "${GIT_USER_GITHUB}"
    - git cms-addpkg PhysicsTools/PatExamples
    - scram b
~~~
{: language-yaml}

The additional two variables that are exported here, `CMSSW_MIRROR` and
`CMSSW_GIT_REFERENCE` are specific to when developing within the CERN
network and can speed up interaction with git, in particular faster
package checkouts.

{% include links.md %}

[cmssw-faq]: http://cms-sw.github.io/faq.html#how-do-i-subscribe-to-github
[custom-gitconfig]: https://stackoverflow.com/a/18330114/11743654