pipeline {
    agent { label 'yocto'}
    stages {
        stage ('dependencies') {
            steps {
                withEnv(['DEBIAN_FRONTEND=noninteractive']) {
                    sh('sudo apt -y update && sudo apt -y upgrade && sudo apt -y install openjdk-11-jdk build-essential gcc-8 g++-8 git bmap-tools chrpath diffstat zstd')
                }
            }
        }
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
                    sh("export TEMPLATECONF=$PWD/meta-greatbear/conf/template-gbeos && \
                    . poky/oe-init-build-env  && \
                    bitbake gbeos-minimal")
                }
            }
        }
        stage("artefacts") {
            steps {
                archiveArtifacts artifacts: 'build/tmp_glibc/deploy/images/**/*.vmdk',
                   allowEmptyArchive: true,
                   fingerprint: true,
                   onlyIfSuccessful: true
                minio bucket: 'gbear-yocto-images/x86vm', 
                   credentialsId: 'minio_gbear-user',
                   targetFolder: 'jenkins-build/',
                   host: 'http://10.60.16.240:9199', 
                   includes: 'build/tmp_glibc/deploy/images/**/*.vmdk'
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