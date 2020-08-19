stage('CI') {
    node {

        git branch: 'jenkins2-course',
            url: 'https://github.com/g0t4/solitaire-systemjs-course'

        // pull dependencies from npm
        // on windows use: bat 'npm install'
        sh 'npm install'

        // stash code & dependencies to expedite subsequent testing
        // and ensure same code & dependencies are used throughout the pipeline
        // stash is a temporary archive
        stash name: 'everything',
              excludes: 'test-results/**',
              includes: '**'

        // test with PhantomJS for "fast" "generic" results
        // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
        sh 'npm run test-single-run -- --browsers PhantomJS'

        // archive karma test results (karma is configured to export junit xml files)
        step([$class: 'JUnitResultArchiver',
              testResults: 'test-results/**/test-results.xml'])

    }
    node('amazon_linux') {
        sh 'ls'
        sh 'rm -rf *'
        unstash 'everything'
        sh 'ls'
    }
}
//parallel integration

// stage('Browser Testing') {
//     parallel chrome: {
//         runTests("Chrome")
//     }, firefox: {
//         runTests("Firefox")
//     }, safari: {
//         runTests("Safari")
//     }
// }
def runTests(browser) {
    node {
        sh 'rm -rf *'
        unstash 'everything'
        sh "npm run test-single-run -- --browsers ${browser}"
        step([$class: 'JUnitResultArchiver',
            testResults: 'test-results/**/test-results.xml'])
    }
}
// good time to create a notify method that will email "Deploy to Staging" or whatever
//input 'Deploy to Staging?'
// limiting concurrency. simultaneous deploys won't happen if multiple pipelines are running.
// Only newest will complete. Rest will be canceled.
lock('solitaireLock') {
    stage('Deploy') {
        node {
            // write a build number to index page so we can see who built this version
            sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
            // deploy to a docker container mapped to 3000 - Dockerfile & docker-compose.yml in repo
            sh '/usr/local/bin/docker-compose up -d --build'
            // send a notify email
        }
    }
}
