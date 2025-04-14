---
title: "Moving to trunk based development"
datePublished: Mon Feb 01 2021 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iwp3m00ynyes18wbd2uu3
slug: moving-to-trunk-based-development
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617382888526/QNvalyKli.jpeg
tags: azure, devops, yaml

---


The project I'm working on is a smaller one. The current release pipeline has a number of specially named branches for each environment. This looks like it was based on GitFlow, but with some oddly specific named branches. So I started thinking about a better way to organise the releases.

Because the project is small, my first instinct was to consolidate on one branch. This looks a lot like [trunk based development](https://trunkbaseddevelopment.com/) and that's because it is. I now only have a master branch from which I deploy to every test environment. With only me as developer and a few business people who test on it, it felt overkill to keep the development and quality assurance environment.

A problem arises when I talk with the business people about release management. They want to test the version in the test environment and when everything looks ok, I can deploy outside of business hours to production. At the moment, I do not have a way to enable new features in the test environment and disable them for the production environment. So I would not be able to continue development without also pushing those changes to production. The same issue arises with hotfixes, I'd need a way to push changes to production, without pushing every experimental feature on the master branch. The best approach would be to use [feature toggles](https://martinfowler.com/articles/feature-toggles.html), but this is not a change I can do overnight.

Queue the only other long lived branch besides the master branch: the production branch. I use feature branches taken from the master branch. When the business gives the green light that everything looks good, I can create a release branch from the master branch. When I have the release branch in place, I can wait until outside working hours before merging the release to production. Basically I create a snapshot from a well tested environment and push it to production. Should bugs in production appear, I can create a hotfix branch from the production branch and merge that back to the test environment for verification.

With this in structure in place, I have only 2 branches: production for the production environment and master for everything else. I can work trunk based in the testing environment and have a dedicated branch for production releases and issues. There is a temporary (and optional) release branch so I have a snapshot of a production release, but that is only for practical reasons.

In the next part, I describe the changes and important remarks I noticed while working towards the end result. This will start from the old implementation with different branches and releases and work towards the solution described above.

So now I just have to change the release pipeline. The first thing I noticed, was that there are a lot of branches that can kick off the single pipeline that is in control of the continuous integration (CI) build, the release build and the deploys to the various environments.


```
trigger:
  - master
  - develop
  - feature/*
  - bugfix/*
  - hotfix/*
  - release/*
``` 

Before a pull request (PR) can be merged into the branches master or develop (both kick off a deploy to a specific environment), it needs to pass a build pipeline. I think the person who set this up, thought that there should be triggers for the CI pipeline to kick in. In Azure DevOps, I can specify a [policy on a branch](https://docs.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops) that needs to run before the PR can complete. This kicks off a separate build process for the policy.

What this in practice does, is when I push to a feature branch (or any other described above), it kicks off a build (described in the release pipeline). Then I start a PR, which also kicks off a build (for the branch policy). I see one build too many here. This does not only cost time as it basically doubles the time for a PR build, it also costs money as the build process is a paid service (after the first 1800 free minutes). It becomes even more wasteful if I need to add changes to the PR. Remember that each push to a branch starts a build and each PR change kicks off a policy build.

Seeing as I do not want multiple branches for each environment (right now, the develop branch goes to the develop environment and the master branch goes to the test and production environments), I can simplify the trigger.

``` 
trigger:
  - master
  - production
``` 

With the simplified triggers, I'm now left with a whole section that is only used for a CI build. As this is not needed for a deployment, I moved this to it's own pipeline. Azure DevOps supports multiple pipelines. I simply create one in the DevOps portal (or maybe through the Azure CLI, I'm not familiar with that) and point it at a yaml file.

``` 
trigger:
  - none

variables:
  DOTNET_SKIP_FIRST_TIME_EXPERIENCE: true
  DOTNET_CLI_TELEMETRY_OPTOUT: true
  solution: src/SolutionFile.sln
  buildPlatform: Any CPU
  buildConfiguration: Release

stages:
  - stage: CI_Build
    displayName: CI build on PR
    jobs:
      - job:
        displayName: CI Build
        pool:
          vmImage: windows-latest
        steps:
          - task: NuGetToolInstaller@1
            displayName: Install NuGet Tooling
          - task: NuGetCommand@2
            displayName: Restore NuGet Packages
            inputs:
              restoreSolution: $(solution)
              feedsToUse: select
              vstsFeed: feed-identifier
          - task: VSBuild@1
            displayName: Build Solution
            inputs:
              solution: $(solution)
              platform: $(buildPlatform)
              configuration: $(buildConfiguration)
          - task: VSTest@2
            displayName: Run Test Suite
            inputs:
              platform: $(buildPlatform)
              configuration: $(buildConfiguration)
``` 

The content of the ci-pipeline.yml contains a simplified build process. I have added two variables to opt out of providing Microsoft with telemetry data about my builds. The [`DOTNET_CLI_TELEMETRY_OPTOUT`](https://docs.microsoft.com/en-us/dotnet/core/tools/telemetry) and [`DOTNET_SKIP_FIRST_TIME_EXPERIENCE`](https://github.com/dotnet/sdk/issues/9927) work together to not send information to Microsoft and speed the build up a little bit. More information can be found in the [Microsoft documentation](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet#environment-variables).

Now that I placed my CI build in a separate pipeline, my original pipeline is already looking cleaner. I'll save you all the refactoring steps, cursing and head scratching (of which there was a lot) and present the finished product. I redacted some parts as they are client specific details, it should not be difficult to figure out what goes where. Let's start with the general release-pipeline.yml file.

``` 
trigger:
  - master
  - production

variables:
  DOTNET_SKIP_FIRST_TIME_EXPERIENCE: true
  DOTNET_CLI_TELEMETRY_OPTOUT: true
  solution: src/SolutionFile.sln
  prodBranch: refs/heads/production

stages:
  - template: pipeline/release-build.yml

  - template: pipeline/deploy.yml
    parameters:
      environment: TEST
      condition: and(succeeded(), not(eq(variables.build.sourceBranch, variables.prodBranch)))

  - template: pipeline/deploy.yml
      parameters:
        environment: PROD
        condition: and(succeeded(), eq(variables.build.sourceBranch, variables.prodBranch))
``` 

The `trigger` and `variables` sections are pretty self explanatory, so lets look at the different stages. I won't display the release-build.yml, but safe to say it looks a lot like the CI pipeline yaml from earlier. I've added a few different steps as I want to build the website and processing services separately. This is necessary to deploy them separately later.

There are two deploys, one to the test environment and one for production. The test deployment runs after a successful build when it's not the production branch and the production deploy only runs after a successful build when it is the production branch. This is the only difference between the deploys.

The deploy.yml in the pipeline folder, contains a stage that calls two job templates: one to deploy the website and one for the processing services. The rest of this template should be pretty straightforward.

``` 
parameters:
  environment: ""
  condition: ""

stages:
 - stage: ${{ parameters.environment }}\_Deploy
    displayName: Deploy To ${{ parameters.environment }}
    dependsOn: Release_Build
    condition: ${{ parameters.condition }}
    pool:
      vmImage: windows-latest
    jobs:
      - template: deploy-website.yml
        parameters:
          environment: ${{ parameters.environment }}
      - template: deploy-processing-services.yml
        parameters:
          environment: ${{ parameters.environment }}
``` 

The website template is just a deployment job that first cleans the target folder, then extracts the new build and finally transforms the web.config.

``` 
parameters:
  environment: ""
  serviceName: "Client.Website"
  artifactPrefix: "Client.Website.Build"

jobs:
  - deployment: Public_Website
    displayName: Install public website
    environment:
      name: ${{ parameters.environment }}
      resourceType: VirtualMachine
      tags: website
    strategy:
      runOnce:
        deploy:
          steps:
            - task: DeleteFiles@1
              displayName: "Delete ${{ parameters.serviceName }}"
              inputs:
                SourceFolder: "C:\\WebSites\\${{ parameters.serviceName }}"
                Contents: "**/*"
            - task: ExtractFiles@1
              displayName: "Extract ${{ parameters.serviceName }}"
              inputs:
                archiveFilePatterns: "$(Pipeline.Workspace)/Mooose/${{ parameters.artifactPrefix }}.$(Build.BuildId).zip"
                cleanDestinationFolder: true
                destinationFolder: "C:\\WebSites\\${{ parameters.serviceName }}"
            - task: FileTransform@2
              displayName: "Transforming $(Environment.Name) web.config"
              inputs:
                folderPath: 'C:\\WebSites\\${{ parameters.serviceName }}'
                xmlTransformationRules: "-transform web.$(Environment.Name).config -xml web.config"
``` 

The deploy-processing-services.yml is quite easy. It installs two agents that process service bus messages. One is for general processing, the other is for pdf generation. This gets its own service as this needs to go fast and can't wait for other general messages. There is an install-service.yml script that contains the powershell to stop, install and start the service. So that is easily reusable as well.

``` 
parameters:
  environment: ""

jobs:
  - deployment: Processing_Services
    displayName: Install processing agents
    environment:
      name: ${{ parameters.environment }}
      resourceType: VirtualMachine
      tags: processing
    strategy:
      runOnce:
        deploy:
          steps:
            - template: install-service.yml
              parameters:
                serviceName: Backend.Agent
                artifactPrefix: client.backend.agent
            - template: install-service.yml
              parameters:
                serviceName: Pdf.Agent
                artifactPrefix: client.pdf.agent
``` 

I don't do a lot of special deployment things, but there is one area where I want to focus on: the environment.tags. I set the deployment to a specific environment name (QA or PROD, from the release-pipeline.yml) on a virtual machine (because they use machines in a local server park) and I use tags to specify which servers I deploy to.

The use of tags allow me to tag one server in the Azure DevOps Environments with the "website" and "processing" tags and everything gets deployed on one server. In the production environment on the other hand, those are several different servers (multiple for the website and one for the services). Yet I can use the same pipeline to deploy to different combinations. Should I ever need more processing servers, I can just add the servers to the Azure Environment, apply the correct tags and the deployment process would know what to do.

With a simplified pipeline, I can spend less time worrying about getting code into the right environment and focus more on getting the features right.
