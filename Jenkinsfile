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
                    sh """
                    SPEC_FILE=~/rpmbuild/SPECS/${PACKAGE_NAME}.spec
                    cat > "$SPEC_FILE" <<EOF
                    Name:           ${PACKAGE_NAME}
                    Version:        ${VERSION}
                    Release:        ${RELEASE}%{?dist}
                    Summary:        Test installation package
                    License:        MIT
                    URL:            https://example.com
                    Source0:        ${TAR_FILE}
                    BuildArch:      noarch

                    %description
                    This package extracts files to ${INSTALL_DIR} when installed.

                    %prep
                    %setup -q

                    %build
                    # No compilation needed

                    %install
                    mkdir -p %{buildroot}${INSTALL_DIR}
                    cp -r * %{buildroot}${INSTALL_DIR}/

                    %files
                    ${INSTALL_DIR}/

                    %changelog
                    * $(date +"%a %b %d %Y") Jenkins Pipeline - ${VERSION}-${RELEASE}
                    - Initial RPM build
                    EOF
                    """
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
