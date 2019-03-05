pipeline {
  /* No idea what I am doing: options? Time trigger? Automatic running of
     cleanup stage if failure on "deployer everything" stage?
  */
  options {
    skipDefaultCheckout() /* skips the clone of automation repo into wrkspc */
    timestamps()
    timeout(time: 30, unit: 'MINUTES', activity: true)
  }

  agent {
    node {
      label "cloud-ccp-ci"
    }
  }

  stages {
    stage('Setup workspace      '){
      steps {
        script {
          sh('''
            set -ex
            export PREFIX=${PREFIX:-'ccpci'}
            export OS_CLOUD=${OS_CLOUD:-'engcloud-cloud-ci'}
            export KEYNAME=${KEYNAME:-'engcloud-cloud-ci'}
            export INTERNAL_SUBNET="${PREFIX}-subnet"
            export ANSIBLE_RUNNER_DIR=~/suse-socok8s-deploy-airship-ashwin
            # Make sure the job has a cloud available, and environemnt
            # vars are properly defined
            cat ~/.config/openstack/clouds.yaml | grep -v password
            env

            # Get started
            mkdir ${WORKSPACE}/ccp/ || true
            pushd ${WORKSPACE}/ccp
                # No need for branchname, as the full periodic job always tests
                # latest master branch
                rm -rf socok8s || true
                git clone --recursive ${ccp_repo} socok8s
                pushd socok8s
                    git checkout ${ccp_branch}
                popd
            popd

          ''')
        }
      }
    }
    stage('Deploy SES'){
      steps {
        script {
          sh('''
            pushd ${WORKSPACE}/ccp/socok8s
              set -ex
              export PREFIX=${PREFIX:-'ccpci'}
              export OS_CLOUD=${OS_CLOUD:-'engcloud-cloud-ci'}
              export KEYNAME=${KEYNAME:-'engcloud-cloud-ci'}
              export INTERNAL_SUBNET="${PREFIX}-subnet"
              export ANSIBLE_RUNNER_DIR=~/suse-socok8s-deploy-airship-ashwin
              ./run.sh deploy_ses
            popd
            ''')
        }
      }
    }
    stage('Deploy CaaSP'){
      steps {
        script {
          sh('''
            pushd ${WORKSPACE}/ccp/socok8s
              export PREFIX=${PREFIX:-'ccpci'}
              export OS_CLOUD=${OS_CLOUD:-'engcloud-cloud-ci'}
              export KEYNAME=${KEYNAME:-'engcloud-cloud-ci'}
              export INTERNAL_SUBNET="${PREFIX}-subnet"
              export ANSIBLE_RUNNER_DIR=~/suse-socok8s-deploy-airship-ashwin
              ./run.sh deploy_caasp
            popd
            ''')
        }
      }
    }
    stage('Deploy CCP Deployer'){
      steps {
        script {
          sh('''
            pushd ${WORKSPACE}/ccp/socok8s
              export PREFIX=${PREFIX:-'ccpci'}
              export OS_CLOUD=${OS_CLOUD:-'engcloud-cloud-ci'}
              export KEYNAME=${KEYNAME:-'engcloud-cloud-ci'}
              export INTERNAL_SUBNET="${PREFIX}-subnet"
              export ANSIBLE_RUNNER_DIR=~/suse-socok8s-deploy-airship-ashwin
            ./run.sh deploy_ccp_deployer
            popd
            ''')
        }
      }
    }
    stage('Enroll CaaSP Workers'){
      steps {
        script {
          sh('''
             pushd ${WORKSPACE}/ccp/socok8s
               export PREFIX=${PREFIX:-'ccpci'}
               export OS_CLOUD=${OS_CLOUD:-'engcloud-cloud-ci'}
               export KEYNAME=${KEYNAME:-'engcloud-cloud-ci'}
               export INTERNAL_SUBNET="${PREFIX}-subnet"
               export ANSIBLE_RUNNER_DIR=~/suse-socok8s-deploy-airship-ashwin
               ./run.sh enroll_caasp_workers
             popd
             ''')
        }
      }
    }
    stage('Setup Airship'){
      steps {
        script {
          sh('''
             pushd ${WORKSPACE}/ccp/socok8s
               export PREFIX=${PREFIX:-'ccpci'}
               export OS_CLOUD=${OS_CLOUD:-'engcloud-cloud-ci'}
               export KEYNAME=${KEYNAME:-'engcloud-cloud-ci'}
               export INTERNAL_SUBNET="${PREFIX}-subnet"
               export ANSIBLE_RUNNER_DIR=~/suse-socok8s-deploy-airship-ashwin

               ##
               ## export DEPLOYMENT_MECHANISM=kvm
               ##

               export SOCOK8S_DEVELOPER_MODE=True

               ##
               ## export AIRSHIP_BUILD_LOCAL_IMAGES=True
               ##

               ./run.sh setup_airship
             popd
             ''')
        }
      }
    }

  }
  post {
    failure {
      script {
        if (env.hold_instance_for_debug == 'true') {
          echo "You can reach this node by connecting to its floating IP (see openstack server show ccp-ci-launcher) as root user, with the default password. In other words: ssh root@10.86.2.167"
          timeout(time: 9, unit: 'HOURS') {
               input(message: "Waiting for input before deleting the ccp env.")
          }
        }
      }
    }
    cleanup {
      script {
        sh('''
          env
          export PREFIX=${PREFIX:-'ccpci'}
          export OS_CLOUD=${OS_CLOUD:-'engcloud-cloud-ci'}
          export KEYNAME=${KEYNAME:-'engcloud-cloud-ci'}
          export INTERNAL_SUBNET="${PREFIX}-subnet"

          ##
          ## export DEPLOYMENT_MECHANISM=kvm
          ##

          export ANSIBLE_RUNNER_DIR=~/suse-socok8s-deploy-airship-ashwin
          export SOCOK8S_DEVELOPER_MODE=True
          export DELETE_ANYWAY=YES

          ##
          ## export AIRSHIP_BUILD_LOCAL_IMAGES=True
          ##

          pushd ${WORKSPACE}/ccp/
            pushd socok8s
              ./run.sh clean_airship
              ./run.sh teardown
            popd
            rm -rf socok8s
          popd
        ''')
      }
    }
  }
}

