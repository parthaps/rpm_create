pipeline {
    agent any

    environment {
        PACKAGE_NAME = "php844"
        VERSION = "1.0"
        RELEASE = "1"
        INSTALL_DIR = "/var/opt/tools"
        TAR_FILE = "php-8.4.4.tar.gz"
    }

    stages {

        stage('Setup RPM Build Environment') {
            steps {
                script {
                    sh '''
                    rpmdev-setuptree
                    mkdir -p ~/rpmbuild/SOURCES/
                    '''
                }
            }
        }

        stage('Copy Source Files') {
            steps {
                script {
                    sh '''
                    cp ${WORKSPACE}/${TAR_FILE} ~/rpmbuild/SOURCES/
                    '''
                }
            }
        }

        stage('Create Spec File') {
            steps {
                script {
                        sh '''
                    cat > ~/rpmbuild/SPECS/testinstall.spec <<EOF
                    Name:           testinstall
                    Version:        1.0
                    Release:        1%{?dist}
                    Summary:        Test installation package
                    License:        MIT
                    URL:            https://example.com
                    Source0:        testinstall.tar.gz
                    BuildArch:      noarch

                    %description
                    This package extracts files to /var/opt/tools when installed.

                    %prep
                    %setup -q

                    %build
                    # No compilation needed

                    %install
                    mkdir -p %{buildroot}/var/opt/tools
                    cp -r * %{buildroot}/var/opt/tools/

                    %files
                    /var/opt/tools/

                    %changelog
                    * $(date +"%a %b %d %Y") Jenkins Pipeline - 1.0-1
                    - Initial RPM build
                    EOF
                    '''
                }
            }
        }

        stage('Build RPM') {
            steps {
                script {
                    sh '''
                    rpmbuild -ba ~/rpmbuild/SPECS/${PACKAGE_NAME}.spec
                    '''
                }
            }
        }

        stage('Archive RPM') {
            steps {
                archiveArtifacts artifacts: 'rpmbuild/RPMS/noarch/*.rpm', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "RPM build completed successfully!"
        }
        failure {
            echo "RPM build failed!"
        }
    }
}