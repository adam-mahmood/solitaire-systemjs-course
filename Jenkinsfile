stage('CI'){
    node {
    	//generic ckeckout scm because the configuration of repository is normally done outside of tis script
    	checkout scm
        //git branch: 'jenkins2-course', 
            //url: 'https://github.com/g0t4/solitaire-systemjs-course'
    
        // pull dependencies from npm
        // on windows use: bat 'npm install'
        bat 'npm install'
    
        // stash code & dependencies to expedite subsequent testing
        // and ensure same code & dependencies are used throughout the pipeline
        // stash is a temporary archive
        stash name: 'everything', 
              excludes: 'test-results/**', 
              includes: '**'
        
        // test with PhantomJS for "fast" "generic" results
        // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
        bat 'npm run test-single-run -- --browsers PhantomJS'
        
        // archive karma test results (karma is configured to export junit xml files)
        step([$class: 'JUnitResultArchiver', 
              testResults: 'test-results/**/test-results.xml'])
              
    }
    node('windows') {
        bat 'dir'
        bat 'del /s /q .'
        unstash 'everything'
        bat 'dir'
    }    
}
//parallel integration testing
stage('Browser Testing'){
    parallel chrome: {
        runTests("Chrome")
    },firefox: {
        runTests("Firefox")
    }
}

    def runTests(browser){
        node {
            bat 'del /s /q .'
            unstash 'everything'
            bat "npm run test-single-run -- --browsers ${browser}"
            
            // archive karma test results (karma is configured to export junit xml files)
            step([$class: 'JUnitResultArchiver', 
                  testResults: 'test-results/**/test-results.xml'])
                          
        }
    
    node {
        notify("Deploy to staging")
    }
    input 'Deploy to staging'
}

stage('Deploy') {
    //limit concurrency so we dont perform simultaneous deploys
    // and if multiple pipelines are executing, then
    // the newest is only that will be allowed through, rest will be cancelled
    // This locked resource contains the deployment stage as a single concurrency Unit.
    // Only 1 concurrent build is allowed to utilize the deployment at a time.
    // Newer builds are pulled off the queue first. When a build reaches the
    // milestone at the end of the lock, all jobs started prior to the current
    // build that are still waiting for the lock will be aborted
    lock(label: 'Deployment', inversePrecedence: true) {
        node {
            // write build number to index page so we can see this update
            // on windows use: bat "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
            //on linux use: sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
            bat "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
            
            // deploy to a docker container mapped to port 3000
            // on windows use: bat 'docker-compose up -d --build'
            //on linux use: sh 'docker-compose up -d --build'
            bat 'docker-compose up -d --build'
            
            notify 'Solitaire Deployed!'
        }
        milestone()
    }
}
def notify(status){
    emailext (
      to: "adam.mahmood786@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
