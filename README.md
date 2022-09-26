### GitHub runners:
+ Receive automatic updates for the operating system, preinstalled packages and tools, and the self-hosted runner application.
+ Are managed and maintained by GitHub.
+ Provide a clean instance for every job execution.
+ Use free minutes on your GitHub plan, with per-minute rates applied after surpassing the free minutes.

### There are 2 types of runners:
1. Runners managed by GitHub itself 
2. Self-hosted ones, where you could choose between Linux, Windows, and Mac operating systems.)

### Triggering the workflow:
2 actions: push and pull request
But we can also filter these triggers for specific branches or files being changed.

#### Job:
+ Jobs can be large and small, and steps can be grouped into a few or as many jobs as needed, depending on the organization’s needs.
+ Consider a separate runner for each repo, so in that way, if a repo is compromised, it is isolated.
+ Have a job related to the AWS, Azure, or GCP account, meaning that we need access to the AWS need to have a larger job that does all AWS, Azure, or GCP-related tasks.
+ We can also schedule for any valid CRON expression or any other options (comments, issues being open).

### Pre-requisite:
1. Git installed locally
2. Github actions being enabled
3. Create a folder inside your repo starting: .githib/workflows and inside this folder create your file with any name but should end .yaml (f.e main.YAML)

### There are 2 types of workflows:
1. Workflow – can be used within one repository
2. Reusable workflow – possible to reuse the same workflow within the enterprize/orgnization

#### Workflow example

``` 
# This workflow should trigger on regular push actions to topic branches ("feature/*", "bugfix/*", etc.) 
name: Topic branches

on:
  push:
    branches:
      - "bugfix/*"
      - "feature/*"

env:
  SLUG: itinerary-services
  CONTAINER_REGISTRY_URL: here is your url
  DEFAULT_TAG: latest

jobs:
  all:
    name: Compile + Run (simpler) tests + Build and release container image
    runs-on: Linux
    container:
      image: your tagged image 
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Run tests
        run: make test
      - name: Build
        run: make build
      - name: Import secrets
        id: vault
        uses: DevOps/hashicorp-vault-action@v2.4.2
        with:
          url: ${{ secrets.VAULT_NONPROD_URL }}
          method: approle
          roleId: ${{ secrets.NONPROD_GITHUB_ACTION_APPROLE_ROLE_ID }}
          secretId: ${{ secrets.NONPROD_GITHUB_ACTION_APPROLE_SECRET_ID }}
          exportEnv: false
          secrets: |
              devops/data/deployment NX_USER_GITHUB_ACTIONS | CONTAINER_REGISTRY_USER ;
              devops/data/deployment NX_PASS_GITHUB_ACTIONS | CONTAINER_REGISTRY_PASSWORD
```

#### What this workflow will be doing? 
1.	"On push braches” means that this action will be triggered if any changes will be made into `bugfix` and `feature` branches.
2.	The block that starts with env, means that we are setting up our own env with slug, which is `itinerary-services`, also we list the registry (nexus) and will have a default tag attached to it.
3.	In the job block we have several tasks that will be applied in the workflow. 
a) Compile and run a test on Linux with a container image that we mentioned in the env. Later it will checkout and install Java 8 with temurin distribution.  
b) Next step is testing if the Java is installed with a specified command `sbt -v +test` and will be also installing NodeJs, which is always used for runners.
c) import secrets from Vault. We put secrets as variables (VAULT_NONPROD_URL, NONPROD_GITHUB_ACTION_APPROLE_ROLE_ID, NONPROD_GITHUB_ACTION_APPROLE_SECRET_ID) those variables will help us to be authenticated.


### Reusable:

We will have 2 workflows: reusable workflow and calling workflow.

Creating and using reusable workflows brings benefits to an organization by making it simple to parameterize, and reuse across repos, even at an enterprise level.  A reusable workflow should always have a calling workflow, which will “call” basically trigger the reusable workflow, by using its variables and jobs. A reusable workflow can be used by another workflow if both workflows are in the same repository. And if the “calling workflow” will be used in the public repository. 

#### Reusable workflow example:

``` 
name: Main
on:
  workflow_call:
    inputs:
      username:
        required: true
        type: string
    secrets:
      envPAT:
        required: true
jobs:
  reusable_workflow_job:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: octo-org/my-action@v1
        with:
          username: ${{ inputs.username }}
          token: ${{ secrets.envPAT }}
``` 

#### What this workflow will be doing? 
1.	For a workflow to be reusable, it must include workflow_call.
2.	Inputs used for variables and secrets that you would specify inside reusable workflow and calling workflow.
3.	envPAT is an environment secret for production environment.

Helpful link for reusable workflow:

[Reusable GitHub Actions workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows)

[On workflow call and on workflow call inputs](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_callinputs)

[On workflow call secrets](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_callsecrets)


#### The "Calling workflow" example
This workflow only used to call reusable workflow by defining the inputs and outputs for the reusable workflow.

``` 
name: Reuse reusable worklow

on: 
  workflow_dispatch:
  
jobs:
  call-workflow-passing-data:
    uses: octo-org/example-repo/.github/workflows/reusable-workflow.yml@main
    with:
      username: mona
    secrets:
      envPAT: ${{ secrets.envPAT }}
``` 

#### What this workflow will be doing? 
1. It will use the parameters from reusable workflow to call it
2. in the `uses` you must put the whole path to the reusable workflow


### How to write your first workflow:

[Start writing GitHub Actions workflow](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)

[How to setup my first workflow](https://docs.github.com/en/actions/quickstart)


