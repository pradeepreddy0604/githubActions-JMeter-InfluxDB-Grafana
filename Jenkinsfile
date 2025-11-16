pipeline {
    agent any

    environment {
        JMETER_DOCKER = "justb4/jmeter:5.6.3"
    }

    parameters {
        string(name: 'JMETER_SCRIPT', defaultValue: 'Scripts/s.jmx', description: 'JMeter test plan file path')
        string(name: 'PROTOCOL', defaultValue: 'HTTP', description: 'Protocol selection of the script')   
        string(name: 'URL_ADDRESS', defaultValue: 'localhost', description: 'Base URL for the test (JPetStore)')
        string(name: 'CSV_DATA_SET_CONFIG', defaultValue: 'Scripts/users.csv', description: 'CSV file path (relative to repo root)')
        string(name: 'CSV_DATA_SET_CONFIG_HEADERS', defaultValue: 'user', description: 'CSV headers (comma-separated)')
        string(name: 'NUMBER_OF_THREADS', defaultValue: '10', description: 'Virtual users count')
        string(name: 'DURATION', defaultValue: '60', description: 'Test duration seconds')
        string(name: 'PORT', defaultValue: '8080', description: 'PORT selection for the script')         
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Run JMeter Test') {
            steps {
                sh '''
                echo "JMETER_SCRIPT=${JMETER_SCRIPT}"
                echo "URL_ADDRESS=${URL_ADDRESS}"
                echo "PROTOCOL=${PROTOCOL}"                
                echo "CSV_DATA_SET_CONFIG=${CSV_DATA_SET_CONFIG}"
                echo "CSV_DATA_SET_CONFIG_HEADERS=${CSV_DATA_SET_CONFIG_HEADERS}"
                echo "NUMBER_OF_THREADS=${NUMBER_OF_THREADS}"
                echo "DURATION=${DURATION}"
                echo "PORT=${PORT}"
                JMX_FILE="/jmeter/$(basename "${JMETER_SCRIPT}")"
                CSV_FILE="/jmeter/$(basename "${CSV_DATA_SET_CONFIG}")"

                docker run --rm \
                  --network=host \
                  -v "$PWD/Scripts:/jmeter" \
                  "${JMETER_DOCKER}" \
                  -n -t "${JMX_FILE}" \
                  -l /jmeter/results.jtl \
                  -e -o /jmeter/report \
                  -JbaseUrl="${URL_ADDRESS}" \
                  -JPROTOCOL="${PROTOCOL}" \
                  -JPORT="${PORT}" \                  
                  -JcsvFile="${CSV_FILE}" \
                  -JcsvHeaders="${CSV_DATA_SET_CONFIG_HEADERS}" \
                  -Jusers="${NUMBER_OF_THREADS}" \
                  -Jduration="${DURATION}" \
                  -JinfluxdbVersion=2 \
                  -JinfluxdbToken=my-super-token \
                  -JinfluxdbOrganization=perf-org \
                  -JinfluxdbBucket=jmeter \
                  -JinfluxdbUrl=http://localhost:8086
                '''
            }
        }

        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: 'Scripts/results.jtl', allowEmptyArchive: true
                archiveArtifacts artifacts: 'Scripts/report/**', allowEmptyArchive: true
            }
        }
    }
}
