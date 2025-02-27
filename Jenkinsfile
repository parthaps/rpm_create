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
                rpmdev-setuptree
                mkdir -p ~/rpmbuild/SOURCES/
                cp $WORKSPACE/$TAR_FILE ~/rpmbuild/SOURCES/
                '''
            }
        }

        stage('Prepare Spec File') {
            steps {
                sh '''
                cp $WORKSPACE/$SPEC_TEMPLATE ~/rpmbuild/SPECS/$SPEC_FILE

                sed -i "s|__VERSION__|$VERSION|g" ~/rpmbuild/SPECS/$SPEC_FILE
                sed -i "s|__RELEASE__|$RELEASE|g" ~/rpmbuild/SPECS/$SPEC_FILE
                sed -i "s|__DATE__|$(date +"%a %b %d %Y")|g" ~/rpmbuild/SPECS/$SPEC_FILE
                '''
            }
        }

        stage('Build RPM') {
            steps {
                sh '''
                rpmbuild -ba ~/rpmbuild/SPECS/$SPEC_FILE
                '''
            }
        }

        stage('Archive RPM') {
            steps {
                archiveArtifacts artifacts: '~/rpmbuild/RPMS/noarch/*.rpm', fingerprint: true
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