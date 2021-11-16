def imageName = 'stephnangue/movies-store'
def registry = '982039600869.dkr.ecr.eu-central-1.amazonaws.com/stephnangue/movies-store'
def credentials = 'ecr:eu-central-1:jenkins_aws_id'

node('workers'){
    stage('Checkout'){
        checkout scm
    }

     def imageTest= docker.build("${imageName}-test", "-f Dockerfile.test .")

    stage('Tests'){
        parallel(
            'Quality Tests': {
                sh "docker run --rm ${imageName}-test npm run lint"
            },
            'Integration Tests': {
                sh "docker run --rm ${imageName}-test npm run test"
            },
            'Coverage Reports': {
                sh "docker run --rm -v $PWD/coverage:/app/coverage ${imageName}-test npm run coverage-html"
                publishHTML (target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: true,
                    reportDir: "$PWD/coverage",
                    reportFiles: "index.html",
                    reportName: "Coverage Report"
                ])
            }
        )
    }

    stage('Build'){
        docker.build(imageName)
    }

     stage('Push'){
        docker.withRegistry("https://"+"${registry}","${credentials}") {
            docker.image(imageName).push(commitID())

            if (env.BRANCH_NAME == 'develop') {
                docker.image(imageName).push('develop')
            }
        }    
    }

    stage('Deploy'){
        if(env.BRANCH_NAME == 'develop'){
            build job: "watchlist-deployment/${env.BRANCH_NAME}"
        }
    }
}

def commitID() {
    sh 'git rev-parse HEAD > .git/commitID'
    def commitID = readFile('.git/commitID').trim()
    sh 'rm .git/commitID'
    commitID
}