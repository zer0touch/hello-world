node {
    try{
        ws("${JENKINS_HOME}/jobs/${JOB_NAME}/builds/${BUILD_ID}/") {
            withEnv(["GOPATH=${JENKINS_HOME}/jobs/${JOB_NAME}/builds/${BUILD_ID}"]) {
                env.PATH="${GOPATH}/bin:$PATH"
                env.GOROOT="/usr/lib/go"
                
                stage('Checkout'){
                    echo 'Checking out SCM'
                    checkout scm
                }
                
                stage('Pre Test'){
                    echo 'Pulling Dependencies'
                    sh 'go version'
                    sh 'go get -u github.com/golang/lint/golint'
                    sh 'go get github.com/tebeka/go2xunit'
                    
                }
        
                stage('Test'){
                    
                    //List all our project files with 'go list ./... | grep -v /vendor/ | grep -v github.com | grep -v golang.org'
                    //Push our project files relative to ./src
                    sh 'cd $GOPATH && go list ./... | grep -v /vendor/ | grep -v github.com | grep -v golang.org > projectPaths'
                    
                    //Print them with 'awk '$0="./src/"$0' projectPaths' in order to get full relative path to $GOPATH
                    def paths = sh returnStdout: true, script: """awk '\$0="./src/"\$0' projectPaths"""
                    
                    echo 'Vetting'


                    echo 'Linting'
                    sh """cd $GOPATH/src && golint ."""
                    
                    echo 'Testing'
                }
            
                stage('Build'){
                    echo 'Building Executable'
                
                    //Produced binary is $GOPATH/src/cmd/project/project
                    sh """cd $GOPATH/src && go build -o hellworld"""
                }
                env.DOCKER_HOST="tcp://build:2376"
                stage('Build Docker Image'){
                    echo 'Building Executable'

                    //Produced binary is $GOPATH/src/cmd/project/project
                    sh """cd $GOPATH/src && docker build -t helloworld ."""
                    sh 'docker images'
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
