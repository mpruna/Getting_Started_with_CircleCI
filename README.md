# Getting started with `Circle CI`

### What is Ciercle CI

`CircleCI` is Continuous Integration, a development practice which is being used by software teams allowing them to to build, test and deploy applications easier and quicker on multiple platforms.

![IMG](https://github.com/mpruna/Getting_Started_with_CircleCI/blob/master/images/Circle_CI.png)

### What is Continuous Integration?

  - Continuous integration is a software engineering practice in which
isolated changes are immediately tested and reported when they are
added to a larger code base.
  - The goal of Continuous integration is to provide rapid feedback so that if
a defect is introduced into the code base, it can be identified and
corrected as soon as possible.


![IMG](https://github.com/mpruna/Getting_Started_with_CircleCI/blob/master/images/CI_pipeline.png)

### Projects and Builds

This document describes how CircleCI automates builds of your project.
### Overview

After a software repository on GitHub or Bitbucket is authorized and added as a project to circleci.com, every code change triggers a build and automated tests in a clean container or VM configured for your requirements.
### Adding Projects

A CircleCI project shares the name of the associated code repository and is visible on the Projects page of the CircleCI app. Projects are added by using the Add Project button.
### Add Projects Page

Following a project enables a user to subscribe to email notifications for the project build status and adds the project to their CircleCI dashboard.

### Viewing Builds

Your build appears on the Jobs page of the CircleCI app when a new commit is pushed to your repository. If you do not see your jobs building on the Jobs page when you push config changes, check the Workflows tab of the CircleCI app to find out how to update your config to enable builds.

![IMG](https://github.com/mpruna/Getting_Started_with_CircleCI/blob/master/images/circle_builds.png)

### Jobs, Steps, and Workflows:

Jobs Overview

Jobs are a collection of Steps. All of the steps in the job are executed in a single unit which consumes a CircleCI container from your plan while it’s running.

Jobs and Steps enable greater control and provide a framework for workflows and status on each phase of a run to report more frequent feedback. The following diagram illustrates how data flows between jobs. Workspaces persist data between jobs in a single Workflow. Caching persists data between the same job in different Workflow builds. Artifacts persist data after a Workflow has finished.

![IMG](https://github.com/mpruna/Getting_Started_with_CircleCI/blob/master/images/circle_workflows.png)

In 2.0 Jobs can be run using the machine executor which enables reuse of recently used machine executor runs, or the docker executor which can compose Docker containers to run your tests and any services they require, such as databases, or the macos executor.

When using the docker executor the container images listed under the docker: keys specify the containers to start. Any public Docker images can be used with the docker executor

```
version: 2                               #CircleCI V2
jobs:                                    #Every config file must have a job    
  build:                                 
    working_directory: /dockerapp        #Default working directory for the test
    docker:                              #Specify container images for this job, jobs will run in container
      - image: docker:17.05.0-ce-git     #or Docker in Docker/Define outside container (docker with git)   
    steps:                  
      - checkout                         #checkout code from GitHub              
      - setup_remote_docker              #create docker images for deployment, using a special key
      - run:                             #Install docker-compose in `CI` environment through `python pip`
          name: Install dependencies
          command: |
            apk add --no-cache py-pip=9.0.0-r1
            pip install docker-compose==1.15.0
      - run:                            #Spin out containers and Run test inside cotnainers
          name: Run tests
          command: |
            docker-compose up -d
            docker-compose run dockerapp python test.py
```

Example of a build and jobs in `Circle CI`

```
version: 2
jobs:
  build:
    docker:
      - image: circleci/<language>:<version TAG>
    steps:
      - checkout
      - run: <command>
  test:
    docker:
      - image: circleci/<language>:<version TAG>
    steps:
      - checkout
      - run: <command>
workflows:
  version: 2
  build_and_test:
    jobs:
      - build
      - test
```
This example shows a parallel job workflow where the build and test jobs run in parallel to save time.

### Using Workflows to Schedule Jobs

To increase the speed of your software development through faster feedback, shorter reruns, and more efficient use of resources, configure Workflows. This document describes the Workflows feature and provides example configurations in the following sections:

### Overview

A workflow is a set of rules for defining a collection of jobs and their run order. Workflows support complex job orchestration using a simple set of configuration keys to help you resolve failures sooner.

### With workflows, you can:

  - Run and troubleshoot jobs independently with real-time status feedback.
  - Schedule workflows for jobs that should only run periodically.
  - Fan-out to run multiple jobs in parallel for efficient version testing.
  - Fan-in to quickly deploy to multiple platforms.

For example, if only one job in a workflow fails, you will know it is failing in real-time. Instead of wasting time waiting for the entire build to fail and rerunning the entire job set, you can rerun just the failed job.
### States

Workflows may appear with one of the following states:

  - RUNNING: Workflow is in progress
  - NOT RUN: Workflow was never started
  - CANCELLED: Workflow was cancelled before it finished
  - FAILING: A Job in the workflow has failed
  - FAILED: One or more jobs in the workflow failed
  - SUCCESS: All jobs in the workflow completed successfully
  - ON HOLD: A job in the workflow is waiting for approval
  - NEEDS SETUP: A workflow stanza is not included or is incorrect in the config.yml file for this project

![IMG](https://github.com/mpruna/Getting_Started_with_CircleCI/blob/master/images/circle_ci_builds.png)

### Failed jobs due to a deprecated parameter:
![IMG](https://github.com/mpruna/Getting_Started_with_CircleCI/blob/master/images/circle_ci_failed.png)

### Successfull job:
![IMG](https://github.com/mpruna/Getting_Started_with_CircleCI/blob/master/images/circle_ci_success.png)

The following sample .circleci/config.yml file shows the default workflow orchestration with two parallel jobs. It is defined by using the workflows: key named build_and_test and by nesting the jobs: key with a list of job names. The jobs have no dependencies defined, therefore they will run in parallel.

```
version: 2
jobs:
  build:
    docker:
      - image: circleci/<language>:<version TAG>
    steps:
      - checkout
      - run: <command>
  test:
    docker:
      - image: circleci/<language>:<version TAG>
    steps:
      - checkout
      - run: <command>
workflows:
  version: 2
  build_and_test:
    jobs:
      - build
      - test
```

### Sequential Job Execution Example

The jobs run according to configured requirements, each job waiting to start until the required job finishes successfully as illustrated in the diagram

![IMG](https://github.com/mpruna/Getting_Started_with_CircleCI/blob/master/images/sequential_job_execution.png)

The following config.yml snippet is an example of a workflow configured for sequential job execution:

```
workflows:
  version: 2
  build-test-and-deploy:
    jobs:
      - build
      - test1:
          requires:
            - build
      - test2:
          requires:
            - test1
      - deploy:
          requires:
            - test2
```

The dependencies are defined by setting the requires: key as shown. The deploy: job will not run until the build and test1 and test2 jobs complete successfully. A job must wait until all upstream jobs in the dependency graph have run. So, the deploy job waits for the test2 job, the test2 job waits for the test1 job and the test1 job waits for the build job

### Fan-Out/Fan-In Workflow Example

The illustrated example workflow runs a common build job, then fans-out to run a set of acceptance test jobs in parallel, and finally fans-in to run a common deploy job

![IMG](https://github.com/mpruna/Getting_Started_with_CircleCI/blob/master/images/parallel_workflow.png)

```
workflows:
  version: 2
  build_accept_deploy:
    jobs:
      - build
      - acceptance_test_1:
          requires:
            - build
      - acceptance_test_2:
          requires:
            - build
      - acceptance_test_3:
          requires:
            - build
      - acceptance_test_4:
          requires:
            - build
      - deploy:
          requires:
            - acceptance_test_1
            - acceptance_test_2
            - acceptance_test_3
            - acceptance_test_4
```

In this example, as soon as the build job finishes successfully, all four acceptance test jobs start. The deploy job must wait for all four acceptance test jobs to complete successfully before it starts.

### Building a CI/CD pipeline with Docker

One of the benefits of using a CI/CD pipeline is that you are alb to do test on commit-ed code before being pushed into production.

### Testing Code

All code must be tested to ensure that high quality, stable code is being released to the public. Python comes with a testing framework named unittest and I’ll be using that for this post

test.py
```
import unittest
import app

class TestDockerapp(unittest.TestCase):

    def setUp(self):                      # Setup/initializes a test version of our Docker app
        self.app = app.app.test_client()

    def test_save_value(self):            # Method used to test submit function
        response = self.app.post('/', data=dict(submit='save', key='2', cache_value='two'))
        assert response.status_code == 200
        assert b'2' in response.data
        assert b'two' in response.data

    def test_load_value(self):           # Method used to test load function
        self.app.post('/', data=dict(submit='save', key='2', cache_value='two'))
        response = self.app.post('/', data=dict(submit='load', key='2'))
        assert response.status_code == 200
        assert b'2' in response.data
        assert b'two' in response.data

if __name__=='__main__':
    unittest.main()
```

### Implement a CI/CD pipeline

Once your project is set up in the CircleCI platform, any commits pushed upstream will be detected and CircleCI will execute the job defined in your config.yml file.

You will need to create a new directory in the repo’s root and a yaml file within this new directory. The new assets must follow these naming schema - directory: .circleci/ file: config.yml in your project’s git repository. This directory and file basically define your CI/CD pipeline and configuration for the CircleCI platform.

The jobs: key represents a list of jobs that will be run. A job encapsulates the actions to be executed. If you only have one job to run then you must give it a key name build: you can get more details about jobs and builds here.


.circke/config.yml
```
version: 2
jobs:
  build:
    working_directory: /dockerapp
    docker:
      - image: docker:17.05.0-ce-git
    steps:
      - checkout
      - setup_remote_docker
      - run:
          name: Install dependencies
          command: |
            apk add --no-cache py-pip=9.0.0-r1
            pip install docker-compose==1.15.0
      - run:
          name: Run tests
          command: |
            docker-compose up -d
            docker-compose run dockerapp python test.py
      - deploy:
          name: Push application Docker image
          command: |
            docker login -u $DOCKER_HUB_USER_ID -p $DOCKER_HUB_PWD
            docker tag dockerapp_dockerapp $DOCKER_HUB_USER_ID/dockerapp:$CIRCLE_SHA1
            docker tag dockerapp_dockerapp $DOCKER_HUB_USER_ID/dockerapp:latest
            docker push $DOCKER_HUB_USER_ID/dockerapp:$CIRCLE_SHA1
            docker push $DOCKER_HUB_USER_ID/dockerapp:latest
```

The build: key is composed of a few elements:

    docker:
    steps:

The docker: key tells CircleCI to use a Docker executor, which means our build will be executed using Docker containers.

image: circleci/docker:17.05.0.-ce-git

steps:

The steps: key is a collection that specifies all of the commands that will be executed in this build. The first action that happens the - checkout command that basically performs a git clone of your code into the build environment.

The - run: keys specify commands to execute within the build. Run keys have a name: parameter where you can label a grouping of commands. For example name: Run Tests groups the test related actions which helps organize and display build data within the CircleCI dashboard.

Each run block is equivalent to separate/individual shells or terminals. Commands that are configured or executed will not persist in later run blocks.

 `Install dependencies`  `run` command installs the needed package dependencies
 `Run tests` build the container and runs tests from test.py onto `dockerapp`
 `Push application Docker image` tags the app and pushes it to `DockerHub`

### Online Resources:

  - [Circle CI Builds](https://circleci.com/docs/2.0/project-build/#section=getting-started)
  - [Circle CI Jobs/Steps/Workflows](https://circleci.com/docs/2.0/jobs-steps/#section=getting-started)
  - [Circle CI using Workspaces for jobs](https://circleci.com/docs/2.0/workflows/#using-workspaces-to-share-data-among-jobs)
  - [Circle CI building Docker Images](https://circleci.com/docs/2.0/building-docker-images/)
  - [CI/CD pipeline with Docker](https://circleci.com/blog/build-cicd-piplines-using-docker/)
