pipeline{

    agent any

    parameters {
        choice choices: ['chrome', 'firefox'], description: 'Select the browser', name: 'BROWSER'
        choice(choices: ['vendor-portal.xml', 'flight-reservation.xml', 'other-test-suite.xml'], description: 'Select the test suite', name: 'TEST_SUITE')
    }

    environment {
        GRID_COMPOSE_FILE = 'grid.yaml'
        TEST_SUITES_COMPOSE_FILE = 'test-suites.yaml'
        DOCKER_COMPOSE_OPTIONS = '--pull=always'
    }

    stages{

        stage('Start Grid'){
            steps{
                sh "docker-compose -f ${GRID_COMPOSE_FILE} up --scale ${params.BROWSER}=2 -d"
            }
        }

        stage('Run Test'){
            steps{
                  script {
                    def suiteName = params.TEST_SUITE.replaceAll('.*/', '').replaceAll('.xml', '')
                    def testOutputDir = "output/${suiteName}"
                    def threadCount = getThreadCount(suiteName)

                    // sh "TEST_SUITE=${suiteName} THREAD_COUNT=${threadCount} docker-compose -f ${TEST_SUITES_COMPOSE_FILE} up --scale runTest=1 ${DOCKER_COMPOSE_OPTIONS} -v ${params.TEST_SUITE}:/home/selenium-docker/test-output/${suiteName} -v ./${testOutputDir}:/home/selenium-docker/test-output"
                    // sh "TEST_SUITE=${suiteName} THREAD_COUNT=${threadCount} docker-compose -f ${TEST_SUITES_COMPOSE_FILE} up --scale runTest=1 ${DOCKER_COMPOSE_OPTIONS} -v ./${testOutputDir}:/home/selenium-docker/test-output"
                    // sh "TEST_SUITE=${suiteName} THREAD_COUNT=${threadCount} docker-compose -f ${TEST_SUITES_COMPOSE_FILE} up ${DOCKER_COMPOSE_OPTIONS}" 
                    // sh "TEST_SUITE=${suiteName} THREAD_COUNT=${threadCount} docker-compose -f ${TEST_SUITES_COMPOSE_FILE} up ${DOCKER_COMPOSE_OPTIONS} -v /home/selenium-docker/test-suites/${params.TEST_SUITE}:/home/selenium-docker/test-output/${suiteName} -v ./${testOutputDir}:/home/selenium-docker/test-output"
                    // sh "TEST_SUITE=${suiteName} THREAD_COUNT=${threadCount} docker-compose -f ${TEST_SUITES_COMPOSE_FILE} up --pull=always --volume /home/selenium-docker/test-suites/${params.TEST_SUITE}:/home/selenium-docker/test-output/${suiteName} --volume ./${testOutputDir}:/home/selenium-docker/test-output ${DOCKER_COMPOSE_OPTIONS}"
                    sh "TEST_SUITE=${params.TEST_SUITE} THREAD_COUNT=${threadCount} docker-compose -f ${TEST_SUITES_COMPOSE_FILE} up --pull=always"
                    // sh "TEST_SUITE=${suiteName} THREAD_COUNT=${threadCount} docker-compose -f ${TEST_SUITES_COMPOSE_FILE} up --pull=always --volume /home/selenium-docker/test-suites/${params.TEST_SUITE}:/home/selenium-docker/test-output/${suiteName}.xml --volume ./${testOutputDir}:/home/selenium-docker/test-output"

                    // if (fileExists("${testOutputDir}/testng-failed.xml")) {
                    //     error("Failed tests found in ${suiteName}")
                    // }
                }
            }
        }

    }

    post {
        always {
            sh "docker-compose -f ${GRID_COMPOSE_FILE} down"
            sh "docker-compose -f ${TEST_SUITES_COMPOSE_FILE} down"

            archiveArtifacts artifacts: 'output/**/emailable-report.html', followSymlinks: false
        }
    }

}

def getThreadCount(suiteName) {
    // Implement logic to determine thread count based on suiteName if needed
    return 3  // Default value, change as per your requirements
}

def fileExists(filePath) {
    return fileExists(filePath, false)
}

def fileExists(filePath, followSymlinks) {
    return fileExists(filePath, followSymlinks, 1)
}

def fileExists(filePath, followSymlinks, maxDepth) {
    return new File(filePath).exists()
}