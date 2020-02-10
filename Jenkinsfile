node {
    def workspace = pwd()
    
    stage('Application_Build') {
        checkout scm
        bat './mvnw clean package -DskipTests'
    }    
    
    stage('Application_Unit_Test') {        
        bat './mvnw compiler:testCompile surefire:test'
        step([$class: 'JUnitResultArchiver', testResults: "**/surefire-reports/*.xml"])
    }    
    stage('Application_Code_Analysis') {        
        withSonarQubeEnv {
            bat './mvnw sonar:sonar -Dsonar.projectKey=Petclinic_Static_Code_Analysis -Dsonar.projectName=Petclinic_Static_Code_Analysis -PQP1'
        }
    }
    stage('Application_Static_Security_Testing') {        
        withSonarQubeEnv {
            bat './mvnw sonar:sonar -Dsonar.projectKey=Petclinic_SAST -Dsonar.projectName=Petclinic_SAST -PQP2'
        }        
    }
    stage('Application_Deploy') {
        bat "copy ${workspace}\\target\\petclinic.war ${TOMCAT_HOME}\\webapps\\"
    }    
    stage('Application_Dynamic_Security_Testing') {
        script {
            try {
                startZap(host: "127.0.0.1", port: "${OWASP_ZAP_PORT}".toInteger(), timeout: 900, zapHome: "${OWASP_ZAP_HOME}")
                sleep (time:45, unit:"SECONDS")
                runZapCrawler(host: "http://localhost:${TOMCAT_PORT}/petclinic")
            }
            catch(err) {
                echo "ERROR: ${err}"
            }
            finally {
                try {
                    runZapAttack()
                }
                catch(err){
                    echo "ERROR: ${err}"
                }
                finally {
                    archiveZap(failAllAlerts: 1, failHighAlerts: 0, failMediumAlerts: 0, failLowAlerts: 0, falsePositivesFilePath: "zapFalsePositives.json")
                }
            }
        }
    }
}
