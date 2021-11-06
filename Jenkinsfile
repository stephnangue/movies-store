def imageName = 'stephnangue/movies-store'

node(){
    stage('Checkout'){
        checkout scm
    }

    def imageTest= docker.build("${imageName}-test", "-f Dockerfile.test .")

    stage('Quality Tests'){
        sh 'npm run lint'
    }
}