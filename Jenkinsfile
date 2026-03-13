pipeline {
    agent { label 'dev-server' }

    stages {

        stage("Code Clone") {
            steps {
                echo "Code Clone Stage"
                git url: "https://github.com/pritha274/node-todo-cicd.git", branch: "master"
            }
        }

        stage("Code Build & Test") {
            steps {
                echo "Code Build Stage"
                sh "docker build -t node-app ."
            }
        }

        stage("Push To DockerHub") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "dockerHubCreds",
                    usernameVariable: "dockerHubUser",
                    passwordVariable: "dockerHubPass"
                )]) {

                    sh 'echo $dockerHubPass | docker login -u $dockerHubUser --password-stdin'
                    sh "docker image tag node-app:latest ${dockerHubUser}/node-app:latest"
                    sh "docker push ${dockerHubUser}/node-app:latest"
                }
            }
        }

        stage("Deploy") {
            steps {
                sh "docker compose down || true"
                sh "docker compose up -d --build"
            }
        }

        stage("Update K8S manifest & push to Repo") {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerHubCreds',
                        usernameVariable: 'dockerHubUser',
                        passwordVariable: 'dockerHubPass'
                    )]) {

                        sh '''
                        cd k8s/
                        cat Deployment.yaml
                        sed -i "s/32/latest/g" deployment.yaml
                        cat Deployment.yaml

                        git add Deployment.yaml
                        git commit -m "Updated Deployment image tag"
                        git push https://github.com/pritha274/node-todo-cicd.git HEAD:master
                        '''
                    }
                }
            }
        }

    }
}
