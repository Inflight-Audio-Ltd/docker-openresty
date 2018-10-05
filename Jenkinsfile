#!groovy

/*
Author Graham McMillan
Email <graham.mcmillan@inflightdublin.com>
Date 31/01/2018
*/

pipeline {
  agent {
    node {
      label 'docker-builder'
    }
  }

  environment {
      TEST_IMAGE_ARM = 'reg.ifddev.com/arm/openresty:test'
      PROD_IMAGE_ARM = 'reg.ifddev.com/arm/openresty:latest'
      CONTAINER = 'openresty'
  }

  stages {
    stage('Cleanup') {
      steps {
        deleteDir()
      }
    }

    stage('Clone repository') {
      steps {
        checkout scm
      }
    }

    stage('Build ARM image') {
      steps {
        // Runs qemu container to enable building of arm images on x86_64
        sh 'docker run --rm --privileged multiarch/qemu-user-static:register --reset'
		script {
		  docker.withRegistry("https://reg.ifddev.com", "docker-registry-credential") {
            sh 'docker build -t ${TEST_IMAGE_ARM} -f DockerfileARM .'
		  }
		}
      }
    }

    stage('Test ARM image') {
      steps {
        sh 'docker run --rm --name ${CONTAINER} ${TEST_IMAGE_ARM} nginx -v'
      }
    }

    stage('Tag images') {
      steps {
        sh 'docker tag ${TEST_IMAGE_ARM} ${PROD_IMAGE_ARM}'
      }
    }

    stage('Push image') {
      steps {
	    script {
	      docker.withRegistry("https://reg.ifddev.com", "docker-registry-credential") {
            sh 'docker push ${PROD_IMAGE_ARM}'
		  }
		}
      }
    }
  }
}
