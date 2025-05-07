pipeline {
    agent any
    
    tools {
        maven 'Maven'  // This should match the Maven installation name in Jenkins
        jdk 'JDK21'    // This should match the JDK installation name in Jenkins
    }
    
    environment {
        MAVEN_OPTS = '-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                // Using optimized Maven build command
                sh '''
                    mvn clean install -T 4 \
                    -DskipTests=false \
                    -Dmaven.javadoc.skip=true \
                    -Dmaven.source.skip=true \
                    -Dmaven.compiler.useIncrementalCompilation=true \
                    -Dmaven.build.timing=true
                '''
            }
        }
        
        stage('Test') {
            steps {
                // Run tests in parallel
                sh '''
                    mvn test -T 4 \
                    -Dparallel=methods \
                    -DthreadCount=4 \
                    -DforkCount=2 \
                    -DreuseForks=true
                '''
            }
        }
    }
    
    post {
        always {
            // Clean up workspace
            cleanWs()
            
            // Publish test results
            junit '**/target/surefire-reports/*.xml'
            
            // Publish Maven artifacts
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
        }
        
        success {
            echo 'Build completed successfully!'
        }
        
        failure {
            echo 'Build failed!'
        }
    }
} 
