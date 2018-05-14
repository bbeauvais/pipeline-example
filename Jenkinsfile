#!/usr/bin/env groovy
/*
* Jenkins Declarative pipeline are wrapped inside a pipeline{} directive where all the job append.
* Some directive are allowed outside like declaring function or pulling libraries. 
* /!\ External directive can broke BlueOcean UI
*/
pipeline {
  /*
  * The build will build on any agent that Jenkins provide, and the same agent for every step.
  * it's possible to have staged level agent but it costs more in ressource, and the workspace is freed 
  * and rebuild on every stage
  */
  agent any
  /*
  * Tools are configured in Jenkins UI the label is the ID gave to them, Example of tools : Maven, Gradle, Docker, npm...
  * You need to declare them here or have them agent scoped to use them in the pipeline
  * Example : even if the mvn command is in the PATH of the Jenkins user on the server you need to declare the tool here to get to use 
  * maven in the pipeline
  */
  tools {
		maven 'Maven 3'
	}
  /*
  * Options are to specify some custom behavior to the pipeline like : fail after a certain amount of time, specify the number of 
  * result to keep Jenkins, print the timestamp before each line in log etc...
  * Options can be applied on the pipeline or at each stage. You can have a pipeline that fail after 1 hour of running but specific
  * Stage that fail after 15 minutes (can be use to avoid the deployment step to get stuck forever)
  */
  options {
    timeout(time: 1, unit: 'HOURS')
  }
  /*
  * Environment is a set a pipeline scoped variable, a lot are created by default depending on the type of build and the available plugins.
  * The directive environment allow you to add some custom variable, this can also be use at stage level. Those variable can be the result of functions
  * and are not necesseraly plain declared, ex : read an information in a file.
  * /!\ Every environnement variable declare on a stage will disapear once the stage is over
  */
  environment {
    ANY_VAR = 'I have been declare as an environment variable'
  }
  /*
  * The Trigger are the element that should trigger the build, they can be UI defined (and the value here will override any UI set) but setting them here
  * is better for versionning and repoductibility of the Job.
  * 3 options are evalable and presented here
  trigger {
    cron(H *\/4 * * 1-5) Accept a cron expression and trigger the job periodically depending on it
    pollSCM('H *\/4 * * 1-5') Accept a cron expression and trigger a check on the source control if any change have been detected, start the job if yes
                              this approch is not recommended by Jenkins developper as it can stress the infrastructure a lot depending on the configuration
    upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS) Accept a list of other Jenkins project and a threshold, if the jobs declared job enter
                                                                                    in the wanted threshold state then the job is started
  }
  */


  /*
  * Stages is where the pipeline Worflow definition append. The workflow is defined by stage on after another each stages having steps.
  * Steps are defined function by Jenkins and his plugin to perform some action.
  */
  stages {
    // Every stage must have a name
    stage('First stage'){
      environment {
        STAGE_SCOPED_VAR = 'I will die soon'
      }
      // And every stage must have at least one step
      steps {
        // the step sh allow you to run command line
        // printenv here will run print every environment variable for the build
        sh 'printenv'

        // echo step is to print some information
        echo "My scoped variable say : ${STAGE_SCOPED_VAR}"
      }
      // Every stage can have custom steps trigger after the execution no matter what and depending on the result (see post directive at the end for more information)
      post {
        always {
          echo 'I will be run no matter what append'
        }
        failure {
          echo 'I will be run only if the stage fail'
        }
        success {
          echo 'I will be run only if the stage succeed'
        }
      }
    }
    // Second stage on the pipeline
    stage('Parallel steps') {
      // Stage can be executed in parallel for some use case and reduice the total execution time of the pipeline, those three stage will be execute at the same time in parallel
      // Use case example : Integration test on different browser can be executed simultanoulsy
      parallel {
        stage('Parallel 1') {
          steps {
            echo 'Parallel Step 1'
          }
        }
        stage('Parallel 2') {
          steps {
            pwd()
          }
        }
        stage('Parallel 3') {
          steps {
            // Use the Slack notification plugin to send a message to a Slack workspace
            slackSend(message: 'Send from Parallel Step 3', color: 'good')
          }
        }
      }
    }
    stage('Simple stage') {
      // Option at the stage level
      options {
        timeout(time: 15, unit: 'MINUTES') //The stage will go to failure state after 15 minutes
        retry(6) // The retry option allow the stage to be retry if it goes in failure or unstable mode
      }
      steps {
          echo 'Could be print at least 6 times'
      }
    }
    stage('Filtered stage') {
      // A stage can be filtered depending on some criteria to be executed only on some situation.
      // If multiple filter are set they all need to be true for the stage to be executed
      // Exemple : Only execute if the some file have been change, only execute when on specific branch
      when {
        environment name: 'GIT_BRANCH', value : 'feature/feature-1' // Execute this step only if environment variable GIT_BRANCH is master
        expression { return true } // Custom expression that return a boolean, it allow some more complex and out of the box control
        beforeAgent true // Special case of when, with this the expression are evaluated before Jenkins give an agent. It could save some execution time
      }
      steps {
        sh 'curl http://www.google.com'
      }
    }
    stage('Ask the user') {
      options {
        timeout(time: 10, unit: 'MINUTES')
      }
      input {
        message "Should we continue?"
        ok "Yes, we should."
        parameters {
          string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        }
      }
      steps {
        echo "Hi ${PERSON}"
      }
    }
  }
  /*
  * The post directive allow you to specify some post pipeline action that will be executed no matter what append depending on the job result.
  * That directive can also be use on stage level. Post stages are like any other stages and can be parameterized, filtered or implemeent the exact same than others.
  */
  post {
    always {
      echo 'I will always be executed, can be usefull for reporting'
    }
    success {
      echo 'I will be executed at the end if the job is successfull'
    }
    failure {
      echo 'I will be executed at the end if the job fail'
    }
    unstable {
      echo 'I will be executed at the end if the job is unstable (Tests failed, quality gate broke...)'
    }
    aborted {
      echo 'I will be executed at the end if someone interupt the job'
    }
    changed {
      echo 'I will be executed at the end if this job status is different than the previous'
    }
    fixed {
      echo 'I will be exectued at the end if the job status improved from the previous one (was failure and now is success'
    }
    regression {
      echo 'I will be executed at the end of the job status is worst than the previous one (was success and now is failure)'
    }
    // The cleanup stage if special, it's like the always but it is executed after every action possible on the job, it is generally used (as is name step) to clean the workspace
    cleanup {
      // After any operation clean workspace
      deleteDir()
    }
  }
}