pipeline {
    agent {
        docker { image 'lochnair/octeon-buildenv:latest' }
    }

    stages {
        stage('Clean') {
            steps {
               sh 'git reset --hard'
               sh 'git clean -fX'
            }
        }

        stage('Install dependencies') {
            steps {
                sh 'su-exec root apt-get update'
                sh 'su-exec root apt-get -y install bc bison flex'
            }
        }

        stage('Prepare') {
            steps {
                sh 'make ARCH=mips ubnt_er_e1000_defconfig'
            }
        }

        stage('Prepare for out-of-tree builds') {
            steps {
                sh 'make -j5 ARCH=mips CROSS_COMPILE=mips64-octeon-linux- prepare modules_prepare'
                sh 'rm -rf tmp && mkdir tmp'
                sh 'tar --exclude-vcs --exclude=tmp -cf tmp/e1000-ksrc.tar .'
                sh 'mv tmp/e1000-ksrc.tar e1000-ksrc.tar'
            }
        }

        stage('Build') {
            steps {
                sh 'make -j5 ARCH=mips CROSS_COMPILE=mips64-octeon-linux- vmlinux modules'
            }
        }
        
        stage('Archive kernel image') {
            steps {
                sh 'mv -v vmlinux vmlinux.64'
                sh 'tar cvjf e1000-kernel.tar.bz2 vmlinux.64'
                archiveArtifacts artifacts: 'e1000-kernel.tar.bz2', fingerprint: true, onlyIfSuccessful: true
            }
        }
        
        stage('Archive kernel modules') {
            steps {
                sh 'make ARCH=mips CROSS_COMPILE=mips64-octeon-linux- INSTALL_MOD_PATH=destdir modules_install'
                sh 'tar cvjf e1000-modules.tar.bz2 -C destdir .'
                archiveArtifacts artifacts: 'e1000-modules.tar.bz2', fingerprint: true, onlyIfSuccessful: true
            }
        }

        stage('Archive build tree') {
            steps {
                sh 'tar -uvf e1000-ksrc.tar Module.symvers'
                sh 'bzip2 e1000-ksrc.tar'
                archiveArtifacts artifacts: 'e1000-ksrc.tar.bz2', fingerprint: true, onlyIfSuccessful: true
            }
        }
    }
}

