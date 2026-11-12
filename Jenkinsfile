pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        TRIVY_IMAGE = 'aquasec/trivy:0.76.1'
        TRIVY_CACHE = 'trivy-cache'
        DOCKER_VOLUME = 'jenkins_home'

        MICROSERVICE_IMAGE = 'us-central1-docker.pkg.dev/online-boutique-ci/microservices-demo/currencyservice:v0.10.7'

        SCAN_ERRORS = '0'
    }

    stages {

        // ==========================================
        // 1. CLEAN WORKSPACE
        // ==========================================
        stage('Clean Workspace') {
            steps {
                deleteDir()
                echo 'Workspace cleaned successfully.'
            }
        }

        // ==========================================
        // 2. CHECKOUT SCM
        // ==========================================
        stage('Checkout SCM') {
            steps {
                checkout scm
                echo 'Repository checkout completed.'
            }
        }

        // ==========================================
        // 3. SCA SCAN
        // ==========================================
        stage('SCA Scan') {
            steps {
                sh 'mkdir -p reports'

                script {

                    // ----------------------------------
                    // SCA - SECURE
                    // ----------------------------------
                    echo 'Scanning secure dependencies...'

                    def secureStatus = sh(
                        returnStatus: true,
                        script: """
                            docker run --rm \
                              -v ${DOCKER_VOLUME}:/var/jenkins_home \
                              -v ${TRIVY_CACHE}:/root/.cache/ \
                              -w "${env.WORKSPACE}" \
                              ${TRIVY_IMAGE} fs \
                              --scanners vuln \
                              --skip-version-check \
                              --format json \
                              --output reports/sca-secure.json \
                              secure/
                        """
                    )

                    if (secureStatus != 0) {
                        env.SCAN_ERRORS = '1'
                        echo 'ERROR: Secure SCA scan failed.'
                    } else {

                        def secureSarifStatus = sh(
                            returnStatus: true,
                            script: """
                                docker run --rm \
                                  -v ${DOCKER_VOLUME}:/var/jenkins_home \
                                  -w "${env.WORKSPACE}" \
                                  ${TRIVY_IMAGE} convert \
                                  --format sarif \
                                  --output reports/sca-secure.sarif \
                                  reports/sca-secure.json
                            """
                        )

                        if (secureSarifStatus != 0) {
                            env.SCAN_ERRORS = '1'
                            echo 'ERROR: Secure SARIF conversion failed.'
                        }
                    }

                    // ----------------------------------
                    // SCA - UNSECURE
                    // ----------------------------------
                    echo 'Scanning unsecure dependencies...'

                    def unsecureStatus = sh(
                        returnStatus: true,
                        script: """
                            docker run --rm \
                              -v ${DOCKER_VOLUME}:/var/jenkins_home \
                              -v ${TRIVY_CACHE}:/root/.cache/ \
                              -w "${env.WORKSPACE}" \
                              ${TRIVY_IMAGE} fs \
                              --scanners vuln \
                              --skip-version-check \
                              --format json \
                              --output reports/sca-unsecure.json \
                              unsecure/
                        """
                    )

                    if (unsecureStatus != 0) {
                        env.SCAN_ERRORS = '1'
                        echo 'ERROR: Unsecure SCA scan failed.'
                    } else {

                        def unsecureSarifStatus = sh(
                            returnStatus: true,
                            script: """
                                docker run --rm \
                                  -v ${DOCKER_VOLUME}:/var/jenkins_home \
                                  -w "${env.WORKSPACE}" \
                                  ${TRIVY_IMAGE} convert \
                                  --format sarif \
                                  --output reports/sca-unsecure.sarif \
                                  reports/sca-unsecure.json
                            """
                        )

                        if (unsecureSarifStatus != 0) {
                            env.SCAN_ERRORS = '1'
                            echo 'ERROR: Unsecure SARIF conversion failed.'
                        }
                    }
                }
            }
        }

        // ==========================================
        // 4. IMAGE SCAN - UNSECURE
        // ==========================================
        stage('Image Scan - Unsecure') {
            steps {
                script {

                    echo 'Scanning app-unsecure:v1...'

                    def scanStatus = sh(
                        returnStatus: true,
                        script: """
                            docker run --rm \
                              -v /var/run/docker.sock:/var/run/docker.sock \
                              -v ${DOCKER_VOLUME}:/var/jenkins_home \
                              -v ${TRIVY_CACHE}:/root/.cache/ \
                              -w "${env.WORKSPACE}" \
                              ${TRIVY_IMAGE} image \
                              --scanners vuln \
                              --skip-version-check \
                              --format json \
                              --output reports/unsecure.json \
                              app-unsecure:v1
                        """
                    )

                    if (scanStatus != 0) {
                        env.SCAN_ERRORS = '1'
                        echo 'ERROR: Unsecure image scan failed.'
                    } else {

                        def sarifStatus = sh(
                            returnStatus: true,
                            script: """
                                docker run --rm \
                                  -v ${DOCKER_VOLUME}:/var/jenkins_home \
                                  -w "${env.WORKSPACE}" \
                                  ${TRIVY_IMAGE} convert \
                                  --format sarif \
                                  --output reports/unsecure.sarif \
                                  reports/unsecure.json
                            """
                        )

                        if (sarifStatus != 0) {
                            env.SCAN_ERRORS = '1'
                            echo 'ERROR: Unsecure SARIF conversion failed.'
                        }
                    }
                }
            }
        }

        // ==========================================
        // 5. IMAGE SCAN - SECURE
        // ==========================================
        stage('Image Scan - Secure') {
            steps {
                script {

                    echo 'Scanning app-secure:v1...'

                    def scanStatus = sh(
                        returnStatus: true,
                        script: """
                            docker run --rm \
                              -v /var/run/docker.sock:/var/run/docker.sock \
                              -v ${DOCKER_VOLUME}:/var/jenkins_home \
                              -v ${TRIVY_CACHE}:/root/.cache/ \
                              -w "${env.WORKSPACE}" \
                              ${TRIVY_IMAGE} image \
                              --scanners vuln \
                              --skip-version-check \
                              --format json \
                              --output reports/secure.json \
                              app-secure:v1
                        """
                    )

                    if (scanStatus != 0) {
                        env.SCAN_ERRORS = '1'
                        echo 'ERROR: Secure image scan failed.'
                    } else {

                        def sarifStatus = sh(
                            returnStatus: true,
                            script: """
                                docker run --rm \
                                  -v ${DOCKER_VOLUME}:/var/jenkins_home \
                                  -w "${env.WORKSPACE}" \
                                  ${TRIVY_IMAGE} convert \
                                  --format sarif \
                                  --output reports/secure.sarif \
                                  reports/secure.json
                            """
                        )

                        if (sarifStatus != 0) {
                            env.SCAN_ERRORS = '1'
                            echo 'ERROR: Secure SARIF conversion failed.'
                        }
                    }
                }
            }
        }

        // ==========================================
        // 6. IMAGE SCAN - MICROSERVICE
        // ==========================================
        stage('Image Scan - Microservice') {
            steps {
                script {

                    echo "Scanning ${MICROSERVICE_IMAGE}..."

                    def scanStatus = sh(
                        returnStatus: true,
                        script: """
                            docker run --rm \
                              -v /var/run/docker.sock:/var/run/docker.sock \
                              -v ${DOCKER_VOLUME}:/var/jenkins_home \
                              -v ${TRIVY_CACHE}:/root/.cache/ \
                              -w "${env.WORKSPACE}" \
                              ${TRIVY_IMAGE} image \
                              --scanners vuln \
                              --skip-version-check \
                              --format json \
                              --output reports/microservice.json \
                              "${MICROSERVICE_IMAGE}"
                        """
                    )

                    if (scanStatus != 0) {
                        env.SCAN_ERRORS = '1'
                        echo 'ERROR: Microservice image scan failed.'
                    } else {

                        def sarifStatus = sh(
                            returnStatus: true,
                            script: """
                                docker run --rm \
                                  -v ${DOCKER_VOLUME}:/var/jenkins_home \
                                  -w "${env.WORKSPACE}" \
                                  ${TRIVY_IMAGE} convert \
                                  --format sarif \
                                  --output reports/microservice.sarif \
                                  reports/microservice.json
                            """
                        )

                        if (sarifStatus != 0) {
                            env.SCAN_ERRORS = '1'
                            echo 'ERROR: Microservice SARIF conversion failed.'
                        }
                    }
                }
            }
        }

        // ==========================================
        // 7. SECURITY GATE
        // ==========================================
        stage('Security Gate') {
            steps {
                script {

                    def jsonReports = [
                        'reports/sca-secure.json',
                        'reports/sca-unsecure.json',
                        'reports/unsecure.json',
                        'reports/secure.json',
                        'reports/microservice.json'
                    ]

                    def sarifReports = [
                        'reports/sca-secure.sarif',
                        'reports/sca-unsecure.sarif',
                        'reports/unsecure.sarif',
                        'reports/secure.sarif',
                        'reports/microservice.sarif'
                    ]

                    def allReports = jsonReports + sarifReports

                    // Check report completeness
                    def missingReports = allReports.findAll {
                        !fileExists(it)
                    }

                    if (!missingReports.isEmpty()) {
                        echo 'Missing reports:'

                        missingReports.each {
                            echo " - ${it}"
                        }
                    }

                    // Initialize severity counters
                    def totals = [
                        CRITICAL: 0,
                        HIGH: 0,
                        MEDIUM: 0,
                        LOW: 0
                    ]


                    // Read JSON reports
                    jsonReports.each { report ->

                        if (fileExists(report)) {

                            try {

                                def data = readJSON(file: report)

                                def counts = [
                                    CRITICAL: 0,
                                    HIGH: 0,
                                    MEDIUM: 0,
                                    LOW: 0
                                ]

                                (data.Results ?: []).each { result ->

                                    (result.Vulnerabilities ?: []).each { vuln ->

                                        def severity = (
                                            vuln.Severity ?: 'UNKNOWN'
                                        ).toUpperCase()

                                        if (counts.containsKey(severity)) {
                                            counts[severity]++
                                            totals[severity]++
                                        }
                                    }
                                }

                                echo """
                                ==================================
                                REPORT: ${report}
                                ==================================
                                CRITICAL : ${counts.CRITICAL}
                                HIGH     : ${counts.HIGH}
                                MEDIUM   : ${counts.MEDIUM}
                                LOW      : ${counts.LOW}
                                """

                            } catch (Exception e) {

                                env.SCAN_ERRORS = '1'

                                echo "ERROR: Unable to parse ${report}"
                                echo "Reason: ${e.getMessage()}"
                            }
                        }
                    }

                    // Display overall summary
                    echo """
                    ==========================================
                    TOTAL VULNERABILITY FINDINGS
                    ==========================================
                    CRITICAL : ${totals.CRITICAL}
                    HIGH     : ${totals.HIGH}
                    MEDIUM   : ${totals.MEDIUM}
                    LOW      : ${totals.LOW}
                    ==========================================
                    """

                    // Check technical scan errors
                    if (env.SCAN_ERRORS == '1' ||
                        !missingReports.isEmpty()) {

                        error(
                            'SECURITY GATE FAILED: Scan errors or incomplete reports detected.'
                        )
                    }

                    // Apply security threshold
                    if (totals.CRITICAL > 0 || totals.HIGH > 0) {

                        error(
                            'SECURITY GATE FAILED: CRITICAL/HIGH vulnerabilities detected.'
                        )
                    }

                    echo 'SECURITY GATE: PASS'
                    echo 'No CRITICAL or HIGH vulnerabilities detected.'
                }
            }
        }
    }

    // ==========================================
    // 8. POST ACTIONS
    // ==========================================
    post {

        always {

            echo 'Archiving security scan reports...'

            archiveArtifacts(
                artifacts: 'reports/**/*',
                allowEmptyArchive: true,
                fingerprint: true
            )

            echo 'Security scan report archiving completed.'
        }

        success {
            echo 'PIPELINE RESULT: SUCCESS'
        }

        failure {
            echo 'PIPELINE RESULT: FAILURE'
        }
    }
}
