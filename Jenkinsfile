pipeline {
    agent any

    environment {
        JMETER_DOCKER = "justb4/jmeter:5.6.3"
    }

    stages {

        stage('User Input Popup') {
            steps {
                script {
                    // This pauses the build and collects input via the Jenkins web UI
                    def config = input(
                        id: 'UserConfigInput',
                        message: 'Fill the following test configuration and click Start Test:',
                        ok: 'Start Test',
                        submitter: '',
                        parameters: [
                            string(name: 'JMETER_SCRIPT', defaultValue: 'Scripts/s.jmx', description: 'Full path of JMeter script inside repo'),
                            string(name: 'CSV_DATA_SET_CONFIG', defaultValue: 'Scripts/users.csv', description: 'CSV file path to be used in test'),
                            string(name: 'CSV_HEADERS', defaultValue: 'user', description: 'Headers inside CSV file'),
                            string(name: 'NUMBER_OF_THREADS', defaultValue: '10', description: 'How many virtual users'),
                            string(name: 'DURATION', defaultValue: '60', description: 'Duration of test (in seconds)'),
                            string(name: 'PROTOCOL', defaultValue: 'http', description: 'http or https'),
                            string(name: 'HOST', defaultValue: 'jpetstore', description: 'Application host/IP'),
                            string(name: 'PORT', defaultValue: '8080', description: 'Application port'),
                            string(name: 'TEST_TITLE', defaultValue: 'MyLoadTest', description: 'Test title for InfluxDB and Grafana'),
                            string(name: 'OUTPUT_DIR', defaultValue: 'Scripts/report', description: 'Where JMeter HTML report should be stored')
                        ]
                    )

                    // Set Environment Variables from user input
                    env.JMETER_SCRIPT = config['JMETER_SCRIPT']
                    env.CSV_DATA_SET_CONFIG = config['CSV_DATA_SET_CONFIG']
                    env.CSV_HEADERS = config['CSV_HEADERS']
                    env.NUMBER_OF_THREADS = config['NUMBER_OF_THREADS']
                    env.DURATION = config['DURATION']
                    env.PROTOCOL = config['PROTOCOL']
                    env.HOST = config['HOST']
                    env.PORT = config['PORT']
                    env.TEST_TITLE = config['TEST_TITLE']
                    env.OUTPUT_DIR = config['OUTPUT_DIR']
                    
                    echo "📌 User Provided Configuration: HOST=${env.HOST}, PORT=${env.PORT}, USERS=${env.NUMBER_OF_THREADS}"
                }
            }
        }

        stage('Run JMeter Test') {
            steps {
                sh '''
                JMX_FILE="/jmeter/$(basename "${JMETER_SCRIPT}")"
                CSV_FILE="/jmeter/$(basename "${CSV_DATA_SET_CONFIG}")"

                # Use Docker Compose service names for internal communication
                BASE_URL="${PROTOCOL}://jpetstore:8080"

                echo "Running test against: ${BASE_URL}"

                docker run --rm \
                  --network=container:influxdb \
                  -v "$PWD/Scripts:/jmeter" \
                  ${JMETER_DOCKER} \
                  -n -t "${JMX_FILE}" \
                  -l /jmeter/results.jtl \
                  -e -o "/jmeter/report" \
                  -JbaseUrl="${BASE_URL}" \
                  -JcsvFile="${CSV_FILE}" \
                  -JcsvHeaders="${CSV_HEADERS}" \
                  -Jusers="${NUMBER_OF_THREADS}" \
                  -Jduration="${DURATION}" \
                  -JtestTitle="${TEST_TITLE}" \
                  -JinfluxdbVersion=2 \
                  -JinfluxdbToken=my-super-token \
                  -JinfluxdbOrganization=perf-org \
                  -JinfluxdbBucket=jmeter \
                  -JinfluxdbUrl=http://influxdb:8086
                '''
            }
        }
        
        // --- RESTORED ARTIFACTS STAGE ---
        stage('Archive Reports') {
            steps {
                // Archive the raw result file
                archiveArtifacts artifacts: 'Scripts/results.jtl', allowEmptyArchive: true
                // Archive the generated HTML report directory
                archiveArtifacts artifacts: 'Scripts/report/**', allowEmptyArchive: true
            }
        }
    }
}
