pipeline {
    agent {
    	  docker { image 'docker.io/krickwix/ybuild:latest' }
    }
    stages {
        stage('scm') {
            steps {
                withEnv(['LANG="C"']) {
                    sh("git submodule update --init --recursive --jobs 32")
                }
            }
        }
        stage("build") {
            steps {
                withEnv(['LANG="C"']) {
                    sh(". setupenv  && \
                    bitbake gbeos-minimal")
                }
            }
        }
        stage("artefacts") {
            steps {
                archiveArtifacts artifacts: 'build/tmp-glibc/deploy/images/**/*.vmdk',
                   allowEmptyArchive: true,
                   fingerprint: true,
                   onlyIfSuccessful: true
                minio bucket: 'gbear-yocto-images-vm', 
                   credentialsId: 'minio_gbear-user',
                   targetFolder: 'jenkins-build/',
                   host: 'http://10.60.16.240:9199', 
                   includes: 'build/tmp-glibc/deploy/images/**/*.vmdk'
            }
        }
    }
    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}