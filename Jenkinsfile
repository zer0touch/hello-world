node {
    try{
        ws("${JENKINS_HOME}/jobs/${JOB_NAME}/builds/${BUILD_ID}/") {
            withEnv(["GOPATH=${JENKINS_HOME}/jobs/${JOB_NAME}/builds/${BUILD_ID}"]) {
                env.PATH="${GOPATH}/bin:$PATH"
                env.GOROOT="/usr/lib/go"
                
                //Checkout our Code
                stage('Checkout'){
                    echo 'Checking out SCM'
                    checkout scm
                }
                
                //Pre test setup
                stage('Pre Test'){
                    echo 'Pulling Dependencies'
                    sh 'go version'
                    sh 'go get -u github.com/golang/lint/golint'
                    sh 'go get -u github.com/gorilla/mux'
                    //sh 'go get github.com/tebeka/go2xunit'   
                }
        
                //Test stage
                stage('Test'){
                    
                    //Push our project files relative to ./src
                    sh 'cd $GOPATH && go list ./... | grep -v /vendor/ | grep -v github.com | grep -v golang.org > projectPaths'
                    
                    //Print them with 'awk '$0="./src/"$0' projectPaths' in order to get full relative path to $GOPATH
                    def paths = sh returnStdout: true, script: """awk '\$0="./src/"\$0' projectPaths"""

                    echo 'Linting'
                    sh """cd $GOPATH/src && golint ."""

                    echo 'Go tests'
                    sh """cd $GOPATH/src && go test ./..."""
                    
                }

                //Adding Sonar Scanner
                stage('Sonar'){

                    echo 'Running Sonar'
                    sh """cd $GOPATH && sonar-scanner -Dsonar.projectKey=hello-world -Dsonar.sources=./src -Dsonar.host.url=http://build:9000 -Dsonar.login=admin -Dsonar.password=admin"""

                    echo 'Testing'
                }
            
                //Build the executable
                stage('Build'){
                    echo 'Building Executable'
                
                    //Produced binary is $GOPATH/src/cmd/project/project
                    sh """cd $GOPATH/src && go build -o helloworld"""
                }
                
                //Change the docker host
                env.DOCKER_HOST="tcp://build:2376"

                //Build the Docker image
                stage('Build Docker Image'){
                    echo 'Building Docker image'

                    //Produced binary is $GOPATH/src/cmd/project/project
                    sh """cd $GOPATH/src && /usr/bin/docker build -t helloworld:latest ."""
                    sh '/usr/bin/docker images'
                    echo 'Exporting Docker image'
                    sh '/usr/bin/docker save -o helloworld.tar helloworld:latest'
                }
                
                //Change the docker host
                env.DOCKER_HOST="tcp://target:2376"

                //Deploy the container
                stage('Deploy'){
                    echo 'Deploying our artifact'

                    //Produced binary is $GOPATH/src/cmd/project/project
                    sh '/usr/bin/docker load -i helloworld.tar'
                    //Delete our running container
                    sh '/usr/bin/docker rm -f helloworld'
                    //Run our deployment
                    sh '/usr/bin/docker run --name helloworld -d -P -p 8080:8080 helloworld:latest /bin/helloworld'
                }

            }
        }
    }catch (e) {
        // If there was an exception thrown, the build failed
        currentBuild.result = "FAILED"
        
        
    } finally {
        // Success or failure, always send notifications
        
        
        def bs = currentBuild.result ?: 'SUCCESSFUL'
        if(bs == 'SUCCESSFUL'){
            
        }
    }
}
