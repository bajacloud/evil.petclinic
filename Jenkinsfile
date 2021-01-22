node {
    def app

    stage('Clone repository') {
        // Let's make sure we have the repository cloned to our workspace

        checkout scm
    }

    stage('Build image') {
        // This builds the actual image; synonymous to docker build on the command line
node {
    def app

    stage('Clone repository') {
        checkout scm
    }

    stage('Build image') {
        //This builds the actual image; synonymous to docker build on the command line
        app = docker.build("library/evilpetclinic:${env.harbor_version}_build", "--build-arg owner_email=${env.owner_email} --build-arg harbor_version=${env.harbor_version} .")
        echo app.id
    }

    stage('Scan Image and Publish to Jenkins') {
        try {
            prismaCloudScanImage ca: '', cert: '', dockerAddress: 'unix:///var/run/docker.sock', ignoreImageBuildTime: true, image: "library/evilpetclinic:${env.harbor_version}_build", key: '', logLevel: 'debug', podmanPath: '', project: '', resultsFile: 'prisma-cloud-scan-results.json'
        } finally {
            prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
        }
    }

    stage('Scan image with twistcli') {
        withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
            sh 'curl -k -u $TL_USER:$TL_PASS --output ./twistcli https://$TL_CONSOLE/api/v1/util/twistcli'
            sh 'sudo chmod a+x ./twistcli'
            sh "./twistcli images scan --u $TL_USER --p $TL_PASS --address https://$TL_CONSOLE --details library/evilpetclinic:${env.harbor_version}_build"
        }
    }

    stage('Push image') {
        //Finally, we'll push the image with two tags. 1st, the incremental build number from Jenkins, then 2nd, the 'latest' tag.
        try {
            docker.withRegistry('https://harbor-master-test.rbenavente.demo.twistlock.com', 'harbor_credentials') {
                app.push("${env.BUILD_NUMBER}")
                app.push("${env.harbor_version}")
                app.push("latest")
            }
        }catch(error) {
            echo "1st push failed, retrying"
            retry(5) {
                docker.withRegistry('https://harbor-master-test.rbenavente.demo.twistlock.com', 'harbor_credentials') {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("${env.harbor_version}")
                    app.push("latest")
                }
            }
        }
    }
}
    }
        