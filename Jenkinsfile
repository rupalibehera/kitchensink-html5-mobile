#!/usr/bin/groovy
node('redhatdistortion-maven') {
    stage 'Checkout'
    checkout scm

    stage 'Build'
    sh 'mvn clean compile'

    stage 'Run Unit Tests'
    sh 'mvn test'

    stage 'Package'
    sh 'mvn package'

    stage 'Archive artifact'
    archive 'target/*.war'

    stage 'Create Image'
    sh 'oc login https://kubernetes.default/ -u openshift-dev -p devel --insecure-skip-tls-verify=true'
    sh 'oc project kitchensink-dev || oc new-project kitchensink-dev'
    sh 'oc process -f https://github.com/tnozicka/openshift-templates/raw/master/binary-build-template.yaml ' +
       '-v "APP_NAME=kitchensink,BUILDER_IMAGE=registry.access.redhat.com/jboss-eap-7/eap70-openshift,OUTPUT_IMAGESTREAM_TAG=kitchensink:latest" ' +
       '| oc apply -f -'
    sh 'mkdir -p ./artifacts/deployments/ && cp target/*.war ./artifacts/deployments/'
    sh 'oc delete builds -l buildconfig=kitchensink'
    sh 'oc start-build kitchensink --from-dir=./artifacts/ --follow'

    stage 'Deploy to dev'
    sh 'oc process -f deployment-template.yaml -v "APP_NAME=kitchensink-dev,IMAGE_STREAM_NAMESPACE=kitchensink-dev,IMAGE_STREAM_TAG=kitchensink:latest" ' +
       '| oc apply -f -'

    stage 'Deploy to test'
    sh 'oc process -f deployment-template.yaml -v "APP_NAME=kitchensink-test,IMAGE_STREAM_NAMESPACE=kitchensink-dev,IMAGE_STREAM_TAG=kitchensink:latest" ' +
       '| oc apply -f -'

    stage 'Deploy to production'
    sh 'oc policy add-role-to-user system:image-puller system:serviceaccount:kitchensink-prod:default --namespace=kitchensink-dev'
    sh 'oc project kitchensink-prod || oc new-project kitchensink-prod'
    // input 'Confirm deploying to production.'
    sh 'oc process -f deployment-template.yaml -v "APP_NAME=kitchensink,IMAGE_STREAM_NAMESPACE=kitchensink-dev,IMAGE_STREAM_TAG=kitchensink:latest" ' +
       '| oc apply -f -'
}
