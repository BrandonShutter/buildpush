node {
   stage('Preparation') {
       // for display purposes
       echo "Preparing"
   }

   stage('Build') {
       // Build an image for scanning
       sh 'echo "FROM ubuntu:16.04" > Dockerfile'
       sh 'echo "MAINTAINER Brandon Shutter <brandon.p.shutter@nasa.gov>" >> Dockerfile'
       sh 'echo "RUN mkdir -p /tmp/test/dir" >> Dockerfile'
       sh 'docker build --no-cache -t dev/ubuntu:test .'
   }

   stage('Scan') {
       twistlockScan ca: '', cert: '', compliancePolicy: 'warn', \
         dockerAddress: 'unix:///var/run/docker.sock', \
         ignoreImageBuildTime: false, key: '', logLevel: 'true', \
         policy: 'warn', repository: 'dev/ubunty', \
         requirePackageUpdate: false, tag: 'test', timeout: 10
   }

   stage('Publish') {
       twistlockPublish ca: '', cert: '', \
         dockerAddress: 'unix:///var/run/docker.sock', key: '', \
         logLevel: 'true', repository: 'dev/ubuntu', tag: 'test', \
         timeout: 10
   }
}
