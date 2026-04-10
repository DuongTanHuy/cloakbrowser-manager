node {
    // def isProduction = env.GIT_BRANCH && (env.GIT_BRANCH == 'production' || env.GIT_BRANCH == 'origin/production')

    // if (isProduction) {
        def app

        stage('Clone repository') {
            checkout scm
        }

        stage('Build image') {
            app = docker.build("huyduongdn/cloakbrowser-manager-k8s")
        }

        // stage('Test image') {
        //     app.inside {
        //         sh 'echo "Tests passed"'
        //     }
        // }

        stage('Push image') {
            docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                app.push("${env.BUILD_NUMBER}")
            }
        }
        
        stage('Trigger ManifestUpdate') {
            echo "triggering updatemanifestjob"
            build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
        }
    // }
}