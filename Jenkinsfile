pipeline {
    agent any

    environment {
        PACKAGE_NAME = "php844"
        VERSION = "1.0"
        RELEASE = "1"
        INSTALL_DIR = "/var/opt/tools"
        TAR_FILE = "php-8.4.4.tar.gz"
        SPEC_TEMPLATE = "testinstall.spec.template"
        SPEC_FILE = "testinstall.spec"
        RPMBUILD_DIR = "${WORKSPACE}/rpmbuild"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm  // Ensures the spec file is available from Git
            }
        }

        stage('Setup RPM Build Environment') {
            steps {
                sh '''
                rpmdev-setuptree --rmpath ${RPMBUILD_DIR}
                mkdir -p ${RPMBUILD_DIR}/SOURCES/
                cp ${WORKSPACE}/${TAR_FILE} ${RPMBUILD_DIR}/SOURCES/
                '''
            }
        }

        stage('Prepare Spec File') {
            steps {
                sh '''
                cp ${WORKSPACE}/${SPEC_TEMPLATE} ${RPMBUILD_DIR}/SPECS/${SPEC_FILE}

                sed -i "s|__PACKAGE_NAME__|${PACKAGE_NAME}|g" ${RPMBUILD_DIR}/SPECS/${SPEC_FILE}
                sed -i "s|__VERSION__|${VERSION}|g" ${RPMBUILD_DIR}/SPECS/${SPEC_FILE}
                sed -i "s|__RELEASE__|${RELEASE}|g" ${RPMBUILD_DIR}/SPECS/${SPEC_FILE}
                sed -i "s|__INSTALL_DIR__|${INSTALL_DIR}|g" ${RPMBUILD_DIR}/SPECS/${SPEC_FILE}
                sed -i "s|__TAR_FILE__|${TAR_FILE}|g" ${RPMBUILD_DIR}/SPECS/${SPEC_FILE}
                sed -i "s|__DATE__|$(date +"%a %b %d %Y")|g" ${RPMBUILD_DIR}/SPECS/${SPEC_FILE}
                '''
            }
        }

        stage('Build RPM') {
            steps {
                sh '''
                rpmbuild -ba ${RPMBUILD_DIR}/SPECS/${SPEC_FILE} --define "_topdir ${RPMBUILD_DIR}"
                '''
            }
        }

        stage('Archive RPM') {
            steps {
                archiveArtifacts artifacts: '${RPMBUILD_DIR}/RPMS/noarch/*.rpm', fingerprint: true
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