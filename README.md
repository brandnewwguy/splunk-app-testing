# Splunk App Testing

Sample CI/CD pipeline for testing a Splunk app against multiple versions of Splunk in parallel.

## How to Use This Repo

This repo goes along with a .conf20 presentation on Fast, Off-the-Shelf Testing for Splunk Apps. There is a small sample Splunk app, along with a running CI/CD testing and building pipeline using GitHub Actions.

- View the pipeline by going to the [Actions page](https://github.com/splunk/splunk-app-testing/actions)
- Clone/Fork the repo or pull the [pipeline configuration](https://github.com/splunk/splunk-app-testing/blob/main/.github/workflows/pipeline.yml) and run it with your own Splunk app code!
- Use our [Dockerfiles](https://github.com/splunk/splunk-app-testing/tree/main/cicd/dockerfiles) to setup your own pipeline
- See the [Repository Layout](#Repository-Layout) section for further explanation of the directories and files in the repo
- We have also included a [.gitlab-ci.yml](https://github.com/splunk/splunk-app-testing/blob/main/cicd/GitLab/.gitlab-ci.yml) with a corresponding script in the same directory for running the same pipeline using GitLab

## Repository Layout

### .github/workflows

This directory holds all the GitHub Action pipeline configurations. The main one is `pipeline.yml`, which has the following stages:

1. **login:** login to the GitHub Container Registry to be able to access Docker images stored in the registry
1. **appinspect:** Run AppInspect on the checked out repository app code and upload the results for viewing after the pipeline has completed
1. **generate-data:** Use Eventgen to generate test data from sample log files and upload the generated data for use in future pipeline steps
1. **splunk:** Use the [cicd_runner.sh](#cicd_runner.sh) script to bring up a Splunk container (version specified by the pipeline job) and Cypress container to run the integration tests against the Splunk app.

### cicd

**dockerfiles**

This directory contains the dockerfiles necessary for the CICD pipeline. The images built from these dockerfiles should be place in a repository which the CICD pipeline can access. eg. artifactory. In this repo, we use Docker Container Registry.

**samples**

This directory holds sample log files that are used by Eventgen. Add more files here and another code block to `inputs.conf` to get more data into Splunk (different source, target index, etc.). We grabbed a sample `access.sample` file from

**eventgen.conf**

This is the configuration file for Eventgen. It tells Eventgen what sample log data to use and how to re-date it. See the [Eventgen docs](http://splunk.github.io/eventgen/).

**test**

This directory holds our Cypress tests and configuration. The `cypress` directory has a structure expected by the testing framework. All the tests go in `cypress/integration`. Check out [Writing Your First Cypress Test](https://docs.cypress.io/guides/getting-started/writing-your-first-test.html#Add-a-test-file).

`cypress.json` is the Cypress configuration. Check out [How To Configure Cypress](https://docs.cypress.io/guides/getting-started/testing-your-app.html#Step-3-Configure-Cypress).

**cicd_runner.sh**

This script is where a lot of the pipeline runs. There are comments in the script where each step happens, but the basic flow is as follows:

1. Create Splunk container (without starting it)
1. Copy the test data and the sample app into the Splunk container
1. Wait for Splunk to be up and to have data using the Splunk REST API
1. Spin up the Cypress container
1. Copy the tests and configuration into the Cypress container
1. Run the tests
1. Copy the Cypress videos out of the Cypress container so CI/CD can save them
1. Stop the containers and Docker network

### testing_app

This directory holds our sample Splunk app, which includes `app.conf`, the app's configuration file, and a sample dashboard `testing_app/default/data/ui/views/website_activity.xml`.

**inputs.conf**

This config file tells Splunk how to take in the data generated by Eventgen in the pipeline. For more information, check out the [inputs.conf docs](https://docs.splunk.com/Documentation/Splunk/8.0.5/Admin/Inputsconf).

**GitLab/.gitlab-ci.yml**

This files runs a CI pipeline for GitLab. In GitLab CI, there are `stages`, defined at the top of the file, that run in their defined order. In each stage, you can have one or more `jobs`, defined in the yaml blocks in the file.

Each job has a name, a Docker image it runs on, the `stage` it runs in, and other optional arguments, such as `scripts` to run or `artifacts` to keep around. For more information, check out the [GitLab CI docs](https://docs.gitlab.com/ee/ci/).

In this sample pipeline, we have 4 stages:

1. `validate-app`, which validates the Splunk app code by running [App Inspect](https://dev.splunk.com/enterprise/docs/developapps/testvalidate/appinspect/) on it, using the Docker container built from the appinspect Dockerfile in the [dockerfiles directory](#dockerfiles)
1. `generate-data`, which uses the data in the [sample directory](#samples) and [eventgen.conf](#eventgen.conf) to generate recently-dated test data
1. `cypress-tests`, which use the [cicd_runner script](#cicd_runner.sh) to run the Cypress tests against different versions of Splunk in parallel
1. `build-artifacts`, which package up our sample app into a tar to be usable as a Splunk app

Note: To run this pipeline in GitLab, copy the contents of this repo into GitLab and put this file in the root directory.

**GitLab/cicd_runner_gitlab.sh**

This is essentially the same file as [cicd_runner.sh](#cicd_runner.sh), but works with GitLab CI's variables and ecosystem.

## Process Flow

The sequence diagram can help to explain the flow of the pipeline. It is constructed using mermaid and the source files for this diagram are present in the docs/ directory.

![Pipeline Sequence Diagram](/docs/images/flow-seq.png)

## Contact Us

We would love to hear from you on how you are testing your own Splunk apps or questions on how to improve your pipeline. Let us know at **CHANGE**
