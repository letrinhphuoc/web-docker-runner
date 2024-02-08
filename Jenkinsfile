pipeline {

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

    stages {

        stage('Start Grid') {
            steps {
                sh "docker-compose -f ${GRID_COMPOSE_FILE} up --scale ${params.BROWSER}=2 -d"
            }
        }

        stage('Run Test') {
            steps {
                script {
                    def suiteName = params.TEST_SUITE.replaceAll('.*/', '').replaceAll('.xml', '')
                    def testOutputDir = "output/${suiteName}"
                    def threadCount = getThreadCount(suiteName)

                    sh "TEST_SUITE=${params.TEST_SUITE} THREAD_COUNT=${threadCount} docker-compose -f ${TEST_SUITES_COMPOSE_FILE} up --pull=always"
                }
            }
        }

    }

    post {
        always {
            script {
                def suiteName = params.TEST_SUITE.replaceAll('.*/', '').replaceAll('.xml', '')
                def testOutputDir = "output/${suiteName}"
                def threadCount = getThreadCount(suiteName)

                currentBuild.result = currentBuild.resultIsBetterOrEqualTo('UNSTABLE') ? 'SUCCESS' : 'FAILURE'
                def htmlReportUrl = "${JENKINS_URL}job/${JOB_NAME}/${BUILD_NUMBER}/artifact/extent-report/ExtentReport.html"

                // Extract total tests, passes, failures, and skips from the log
                def logContent = currentBuild.rawBuild.getLog(10000).join('\n')
                def totalTests = extractTestCount(logContent, 'Total tests run: (\\d+)')
                def passes = extractTestCount(logContent, 'Passes: (\\d+)')
                def failures = extractTestCount(logContent, 'Failures: (\\d+)')
                def skips = extractTestCount(logContent, 'Skips: (\\d+)')

                slackSend(
                    channel: 'jenkins',
                    attachments: [
                        [
                            color: currentBuild.result == 'SUCCESS' ? 'good' : 'danger',
                            pretext: "*Test Execution Summary*",
                            fallback: "Test Execution Summary: ${params.TEST_SUITE} - ${currentBuild.result}",
                            fields: [
                                [
                                    title: "Test Suite",
                                    value: params.TEST_SUITE,
                                    short: true
                                ],
                                [
                                    title: "Build Result",
                                    value: currentBuild.result,
                                    short: true
                                ],
                                [
                                    title: "HTML Report",
                                    value: "<$htmlReportUrl|View HTML Report>",
                                    short: false
                                ],
                                [
                                    title: "Total Tests",
                                    value: totalTests,
                                    short: true
                                ],
                                [
                                    title: "Passes",
                                    value: passes,
                                    short: true
                                ],
                                [
                                    title: "Failures",
                                    value: failures,
                                    short: true
                                ],
                                [
                                    title: "Skips",
                                    value: skips,
                                    short: true
                                ],
                                [
                                    title: "Pipeline ID",
                                    value: "${BUILD_TAG.replaceAll('jenkins-SELENIUM_DOCKER_RUNNER-', '')}",
                                    short: true
                                ]
                            ]
                        ]
                    ],
                    teamDomain: 'james-automation',
                    token: '5TJfXN1RESkjqyIhNjqFPTwX'
                )
            }
        }
    }
}

def getThreadCount(suiteName) {
    // Implement logic to determine thread count based on suiteName if needed
    return 3  // Default value, change as per your requirements
}

def extractTestCount(logContent, pattern) {
    def matcher = (logContent =~ pattern)
    if (matcher.find()) {
        return matcher.group(1)
    } else {
        return 'N/A'
    }
}
