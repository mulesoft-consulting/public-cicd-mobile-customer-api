pipeline {
  agent {
    label 'bat-builder'
  }
  environment {
    DEPLOY_CREDS = credentials('deploy-anypoint-user')
    MULE_VERSION = '4.1.5'
    BG = "1Platform\\Public\\CI-CD Demo"
    WORKER = "Micro"
    APPNAME = "nto-mobile-experience"
  }
  stages {
    stage('Build') {
      steps {
            sh 'mvn -B -U -e -V clean -DskipTests package'
      }
    }

    stage('Test') {
      steps {
	      withMaven(
          mavenSettingsConfig: 'public-maven-config.xml') {
            sh "mvn -B -Dmule.env=dev test"
          }
      }
      post {
        always {
          publishHTML (target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'target/site/munit/coverage',
                        reportFiles: 'summary.html',
                        reportName: "Code coverage"
                    ]
                  )
        }
      }
    }

    stage('Deploy Development') {
      environment {
        ENVIRONMENT = 'Development'
        APP_NAME = 'dev-${APPNAME}'
      }
      steps {
            sh 'mvn -U -V -e -B -DskipTests deploy -DmuleDeploy -Dmule.version=$MULE_VERSION -Danypoint.username=$DEPLOY_CREDS_USR -Danypoint.password=$DEPLOY_CREDS_PSW -Dcloudhub.app=$APP_NAME -Dcloudhub.environment=$ENVIRONMENT -Dcloudhub.bg="$BG" -Dcloudhub.worker=$WORKER -Denv.name=dev'
      }
    }

    stage('Integration Test') {
        steps {
            sh 'sed -i -e "s/url:.*$/url: \'http:\\/\\/dev-${APPNAME}.us-e2.cloudhub.io\\/api\',/g" integration-tests/config/devx.dwl'
            sh 'bat integration-tests --config=devx'
        }
        post {
          always {
            publishHTML (target: [
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: '/tmp',
                            reportFiles: 'index.html',
                            reportName: "Integration Test",
                            includes: '**/index.html'
                        ]
                      )       
          }
        }
    }
    stage('Deploy Production') {
        environment {
          ENVIRONMENT = 'Production'
          APP_NAME = '${APPNAME}'
        }
        steps {
              sh 'mvn -U -V -e -B -DskipTests deploy -DmuleDeploy -Dmule.version=$MULE_VERSION -Danypoint.username=$DEPLOY_CREDS_USR -Danypoint.password=$DEPLOY_CREDS_PSW -Dcloudhub.app=$APP_NAME -Dcloudhub.environment=$ENVIRONMENT -Dcloudhub.bg="$BG" -Dcloudhub.worker=$WORKER -Denv.name=prod'
        }
  }

  stage('Install Functional Monitoring') {
      environment {
          TARGET="75c403a6-8054-43ec-b611-63b9efff820d"
      }
      steps {
            sh 'sed -i -e "s/name:.*$/name: \"${APPNAME}_$(date +%Y%m%d%H%M%S)\"/g" integration-tests/bat.yaml'
            sh 'sed -i -e "s/url:.*$/url: \'http:\\/\\/${APPNAME}.us-e2.cloudhub.io\\/api\',/g" integration-tests/config/devx.dwl'
            sh 'bat schedule create --name=$APPNAME --target=$TARGET integration-tests'
      }
  }
}
  post {
      always {
       step([$class: 'hudson.plugins.chucknorris.CordellWalkerRecorder'])
      }
  }
  tools {
    maven 'M3'
  }
}
