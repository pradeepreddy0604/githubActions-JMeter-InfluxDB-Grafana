pipeline {
    agent any

    environment {
        JMETER_DOCKER = "justb4/jmeter:5.6.3"
    }

    parameters {
        string(name: 'JMETER_SCRIPT', defaultValue: 'Scripts/s.jmx', description: 'JMeter test plan file path')
        string(name: 'URL_ADDRESS', defaultValue: 'http://jpetstore:8080', description: 'Base URL for the test')
        string(name: 'CSV_DATA_SET_CONFIG', defaultValue: 'Scripts/users.csv', description: 'CSV file path')
        string(name: 'CSV_DATA_SET_CONFIG_HEADERS', defaultValue: 'user', description: 'CSV headers')
        string(name: 'NUMBER_OF_THREADS', defaultValue: '10', description: 'Virtual users count')
        string(name: 'DURATION', defaultValue: '60', description: 'Test duration seconds')
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Run JMeter Test') {
            steps {
                sh """
                docker run --rm                   -v $PWD/Scripts:/jmeter                   -v $PWD:/data                   ${JMETER_DOCKER}                   -n -t /jmeter/$(basename ${JMETER_SCRIPT})                   -l /jmeter/results.jtl                   -e -o /jmeter/report                   -JbaseUrl=${URL_ADDRESS}                   -JcsvFile=${CSV_DATA_SET_CONFIG}                   -JcsvHeaders=${CSV_DATA_SET_CONFIG_HEADERS}                   -Jusers=${NUMBER_OF_THREADS}                   -Jduration=${DURATION}                   -JinfluxdbVersion=2                   -JinfluxdbToken=my-super-token                   -JinfluxdbOrganization=perf-org                   -JinfluxdbBucket=jmeter                   -JinfluxdbUrl=http://influxdb:8086
                """
            }
        }

        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: 'Scripts/results.jtl'
                archiveArtifacts artifacts: 'Scripts/report/**'
            }
        }
    }
}
