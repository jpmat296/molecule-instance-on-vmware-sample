import groovy.transform.Field
import org.jenkinsci.plugins.workflow.steps.FlowInterruptedException

stage('root') {
  try {
    parallel lin: {
      stage('lin') {
        node('vsphere') {
          sleep 1
          executeToxEnvironment('py38-lin')
        }
      }
    },
    win: {
      stage('win') {
        node('vsphere') {
          sleep 30
          executeToxEnvironment('py38-win')
        }
      }
    }
  } catch (e) {
    currentBuild.result = 'FAILURE'
    notifyFailed()
    throw e
  } finally {
    println 'currentBuild.result: ' + currentBuild.result
    if (currentBuild.result != 'FAILURE') {
      notifySuccessful()
    }
  }
}

def executeToxEnvironment(String toxEnv) {
  checkout([$class: 'GitSCM', branches: [[name: env.BRANCH_NAME]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: "molecule-instance-on-vmware-sample"]],                submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'f37e5623-35de-4909-ba30-72a3dad7a582', url: "git@github.com:jpmat296/molecule-instance-on-vmware-sample.git"]]])
  stage('tox') {
    timestamps {
      try {
        stage('test') {
          sh """#!/bin/bash
            set -xe
            bash ~/bin/vms_destroy.sh || true
            source /usr/local/pyenv/.pyenvrc
            cd molecule-instance-on-vmware-sample
            { tox --workdir .tox -e ${toxEnv} | tee ${toxEnv}.log; } || true
            grep "Failed to connect to the host via ssh" ${toxEnv}.log
          """
        }
      } finally {
        stage('destroy') {
          sh """#!/bin/bash
            set -xe
            bash ~/bin/vms_destroy.sh || true
          """
        }
      }
    }
  }
}

def notifySuccessful() {
  emailext (
      subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",

body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
      to: 'jp@matsusoft.com'
  )
}

def notifyFailed() {
  emailext (
      subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
      to: 'jp@matsusoft.com'
  )
}
