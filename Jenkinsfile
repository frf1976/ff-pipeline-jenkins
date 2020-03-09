def mergeBack = false
def targetBranch = 'master'
def workspace_dir = '/home/ubuntu/workspace/workspace/odm-workflow_master'

node {
  stage('checkout') {
    deleteDir()
    checkout scm
    if(env.BRANCH_NAME =~ "ready/*") {
      mergeBack = true
      sshagent (credentials: ['drbosse']) {
        sh "git config remote.origin.fetch \"+refs/heads/*:refs/remotes/origin/*\""
        sh "git fetch origin ${targetBranch}"
        sh "git checkout -b ${targetBranch} origin/${targetBranch}"
        sh "git merge --ff-only origin/${env.BRANCH_NAME}"
      }
    }
  }
  stage('Checkout external ansible-lint project') {
    sh "git clone https://github.com/ansible/ansible-lint.git"
  }
  stage('test') {
    docker.image('frf1976/odm').withRun() {
      sh "docker run -v ${workspace_dir}/ansible-lint:/usr/src/odm frf1976/odm examples/example.yml" 
    }
  }
  stage('push ?'){
    if(mergeBack){
      sshagent (credentials: ['2d19facc-27df-46fa-8b54-37e6b5c31f14']) {
        sh "git push origin ${targetBranch}"
        sh "git push origin :${env.BRANCH_NAME}"
        currentBuild.description = "Merged ${env.BRANCH_NAME}"
      }
    } else {
      echo "Nothing to push to the server"
    }
  }
}
