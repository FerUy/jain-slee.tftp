pipeline {
    agent any

	tools {
	  jdk 'JDK 11'
		maven 'Maven_3.8.5'
	}
	parameters { string(name: 'JSLEE_TFTP_VERSION', defaultValue: '7.2.0', description: 'The major version for JAIN SLEE TFTP version') }

  environment{
      TFTP_BUILD_VERSION = "${params.JSLEE_TFTP_VERSION}-${BUILD_NUMBER}"
  }

	stages{
		stage("Build ") {
			steps {
			    sh 'java -version'
          script {
              TFTP_BUILD_VERSION = "${params.JSLEE_TFTP_VERSION}-${BUILD_NUMBER}"
                  currentBuild.displayName = "#${TFTP_BUILD_VERSION}"
                  currentBuild.description = "JAIN SLEE TFTP (${env.BRANCH_NAME})"
              }
				      echo "Building JAIN SLEE TFTP version (#${TFTP_BUILD_VERSION})"
              sh "mvn versions:set -DgenerateBackupPoms=false -DnewVersion=${TFTP_BUILD_VERSION} clean install -DskipTests"
				      sh 'mvn clean install -DskipTests'
              echo "Maven build completed."
            }
        }

		stage("Release") {
			steps {
				echo "Building a wildfly release version of #${TFTP_BUILD_VERSION}"
                sh 'find . -type d -name \\"target\\" -exec r -r {} +'
                sh "mvn versions:set -DnewVersion=${TFTP_BUILD_VERSION} clean install -Prelease -Drelease.dir=../../../${TFTP_BUILD_VERSION} -Dmaven.test.skip=true"
                echo "Building wildfly version completed."
            }
        }
	}
}
