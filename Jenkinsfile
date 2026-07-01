pipeline {
	agent any
	tools {
        jdk 'jdk-11'
        maven 'maven-3.9.12'
    }
	options {
    	buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '10'))
  	}
	parameters {
		string(name: 'JSLEE_TFTP_VERSION', defaultValue: '7.2.0', description: 'The major version for JAIN SLEE TFTP')
	}
	environment {
        TFTP_BUILD_VERSION = "${params.JSLEE_TFTP_VERSION}-${BUILD_NUMBER}"
    }
	stages {
		stage("Build") {
			steps {
				script {
                    TFTP_BUILD_VERSION = "${params.JSLEE_TFTP_VERSION}-${BUILD_NUMBER}"
                    currentBuild.displayName = "#${TFTP_BUILD_VERSION}"
                    currentBuild.description = "JAIN SLEE TFTP (${env.BRANCH_NAME})"
                }
                sh "mvn clean install -Dmaven.test.skip=true"
			}
		}
		stage('Set Version') {
			steps {
				sh "mvn versions:set -DgenerateBackupPoms=false -DnewVersion=${TFTP_BUILD_VERSION}"
                sh "mvn versions:commit"
			}
		}
		stage("Release") {
			steps {
                sh "mvn clean install -Prelease -Drelease.dir=../../../${TFTP_BUILD_VERSION} -Dmaven.test.skip=true"
			}
		}
		stage('Zip Resources') {
			steps {
				dir("${TFTP_BUILD_VERSION}/resources") {
					sh "zip -r tftp.zip tftp-server"
					sh 'rm -rf tftp-server'
				}
			}
		}
		stage('Save Artifacts') {
            steps {
                archiveArtifacts artifacts: "${TFTP_BUILD_VERSION}/", followSymlinks: false, onlyIfSuccessful: true
            }
		}
        stage('Push to Repo') {
            when{anyOf {branch 'master'; branch 'release'}}
			steps {
                sh "mkdir -p /var/www/html/NAIKERI/jain-slee.tftp/${TFTP_BUILD_VERSION}/"
                sh "cp -r ${TFTP_BUILD_VERSION}/ /var/www/html/NAIKERI/jain-slee.tftp/${TFTP_BUILD_VERSION}/"
				sh "rm -rf ${TFTP_BUILD_VERSION}"
			}
		}
    }
	post {
		success { echo "JAIN-SLEE TFTP successfully built" }
		failure { echo "Building JAIN-SLEE TFTP failed" }
	}
}
