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
        ARTIFACTORY_URL = "https://testartif.jfrog.io/artifactory/thirdparty-generic-local/php-8.4.4.tar.gz"
        ARTIFACTORY_UPLOAD_URL = "https://testartif.jfrog.io/artifactory/api/generic/created_rpm"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm  // Ensures the spec file is available from Git
            }
        }

        stage('Setup RPM Build Environment') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'artiftoken', usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    sh '''
                    mkdir -p ${RPMBUILD_DIR}
                    rpmdev-setuptree --rmpath ${RPMBUILD_DIR}
                    mkdir -p ${RPMBUILD_DIR}/SOURCES/
                    mkdir -p ${RPMBUILD_DIR}/SPECS/
                    curl -u ${ARTIFACTORY_USER}:${ARTIFACTORY_PASSWORD} -o ${RPMBUILD_DIR}/SOURCES/${TAR_FILE} ${ARTIFACTORY_URL}
                    '''
                }
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

        stage('Upload RPM to Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'artiftoken', usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    sh '''
                    for rpm in ${RPMBUILD_DIR}/RPMS/noarch/*.rpm; do
                        curl -u ${ARTIFACTORY_USER}:${ARTIFACTORY_PASSWORD} -T $rpm ${ARTIFACTORY_UPLOAD_URL}/$(basename $rpm)
                    done
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "RPM build and upload completed successfully!"
        }
        failure {
            echo "RPM build or upload failed!"
        }
    }
}