node {
    def isProduction = env.GIT_BRANCH && (env.GIT_BRANCH == 'production' || env.GIT_BRANCH == 'origin/production')

    if (isProduction) {
        def app

        stage('Clone repository') {
            checkout scm
        }

        stage('Build image') {
            app = docker.build("huyduongdn/cloakbrowser-manager-k8s")
        }

        stage('Push image') {
            docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                app.push("${env.BUILD_NUMBER}")

            }
        }

        stage('Trigger ManifestUpdate') {
            echo "triggering updatemanifestjob"
            build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
        }
    } else {
        currentBuild.result = 'SUCCESS'
        echo "Bỏ qua build vì đây không phải nhánh production (Branch: ${env.GIT_BRANCH})"
    }
}

node {
    // Checkout để Jenkins lấy được thông tin branch chính xác từ SCM
    stage('Clone repository') {
        checkout scm
    }

    // Lấy tên branch hiện tại
    def branchName = env.BRANCH_NAME ?: sh(script: "git rev-parse --abbrev-ref HEAD", returnStatus: false, returnStdout: true).trim()
    
    def isProduction = (branchName == 'production' || branchName == 'origin/production')

    if (isProduction) {
        def app

        stage('Build image') {
            app = docker.build("huyduongdn/cloakbrowser-manager-k8s")
        }

        stage('Push image') {
            docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                app.push("${env.BUILD_NUMBER}")
                app.push("latest") // Nên push thêm tag latest để dễ quản lý
            }
        }
        
        stage('Trigger ManifestUpdate') {
            echo "Triggering updatemanifestjob with tag: ${env.BUILD_NUMBER}"
            build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
        }
    } else {
        currentBuild.result = 'SUCCESS'
        echo "Bỏ qua build vì đây không phải nhánh production (Branch: ${branchName})"
    }
}