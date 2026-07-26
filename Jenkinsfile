pipeline {
    agent any

    stages {
        stage('Build Jar') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Image') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                bat 'docker build -t=sundeepkmr/selenium:latest .'
            }
        }

        stage('Start Grid') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                bat 'docker compose -f grid.yaml up -d'
            }
        }

        stage('Run Test') {
            steps {
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    bat 'docker compose -f test-suites.yaml up'
                }
            }
        }
        
        stage('Evaluate Results') {
            when {
                expression { currentBuild.result == 'FAILURE' }
            }
            steps {
                error "Pipeline is failing because a previous stage encountered errors."
            }
        }
    }

    post {
        // This block will now ONLY run if the entire pipeline completes successfully
        success {
            bat 'docker compose -f grid.yaml down'
            bat 'docker compose -f test-suites.yaml down --remove-orphans'
            archiveArtifacts artifacts: 'output/flight-reservation/emailable-report.html', followSymlinks: false
            archiveArtifacts artifacts: 'output/vendor-portal/emailable-report.html', followSymlinks: false
            cleanWs()
        }
    }
}
