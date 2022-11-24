#!groovy
import java.text.SimpleDateFormat

def dateFormat = new SimpleDateFormat("yyyyMMddHHmm")
def date = new Date()
def timestamp = dateFormat.format(date).toString()
def CORREOS = "mrubidem@choucairtesting.com"

pipeline {
    agent any
    stages {
        stage('Obtener Fuentes')
                {
                    steps
                            {
                                checkout([$class: 'GitSCM', branches: [[name: "master"]],
                                          wdoGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [
                                        [credentialsId: "githubMiguel", url: "https://github.com/mimarumo25/ProyectoBase-Screemplay"]
                                ]])
                            }
                }

        stage('SonarQube analysis')
                {
                    steps {
                        script {
                            scannerHome = tool 'SonarQubeScanner'
                            //mismo nombre del servidor configurado en las Global Tools Jenkins
                        }
                        withSonarQubeEnv('sonarQube')//mismo nombre del servidor configurado en la configuracion del sistema jenkins
                                {
                                    bat 'sonar-scanner'
                                }
                    }
                }

        stage("Quality Gate") {
            steps {
                script {
                         timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
                         def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                         if (qg.status != 'OK') {
                                error "Pipeline aborted due to quality gate failure: ${qg.status}"
                            }
                         }
                }
            }
        }

        stage('Ejecutar Pruebas') {
            steps {
                script {
                    try {
                        //bat ("gradle clean test -DRunner=\"${Runner}\" aggregate") //Ejecución en agente Windows con parametro jenkins
                        //sh ("gradle clean test -DRunner=\"${Runner}\" aggregate") //Ejecución en agente Linux con parametro jenkins
                        bat("gradle clean test aggregate") //Ejecución en agente windows sin parametro jenkins
                        echo 'Test Ejecutados sin Fallo'
                        currentBuild.result = 'SUCCESS'
                    }
                    catch (ex) {
                        echo 'Test Ejecutados con Fallo'
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Generar evidencia'){
            steps
                    {
                        script
                                {
                                    try
                                    {
                                        bat  " rename \"${WORKSPACE}\\target\" serenity_${timestamp}"
                                        echo 'Backup de evidencias realizado con exito'

                                        publishHTML([
                                                allowMissing: false,
                                                alwaysLinkToLastBuild: true,
                                                keepAll: true,
                                                reportDir: "${WORKSPACE}//serenity_${timestamp}/site/serenity",
                                                reportFiles: 'index.html',
                                                reportName: 'Evidencias Serenity Demo ',
                                                reportTitles: 'Proyecto Serenity Demo SCREEMPLAY'
                                        ])
                                        echo 'Reporte Html realizado con exito'
                                    }
                                    catch(e)
                                    {
                                        echo 'No se realizo el Backup de evidencias'
                                        publishHTML([allowMissing: false,
                                         alwaysLinkToLastBuild: true,
                                         keepAll: true,
                                         reportDir: "${WORKSPACE}//target/serenity_${timestamp}",
                                         reportFiles: 'index.html',
                                         reportName: 'Evidencias Serenity Demo ',
                                         reportTitles: 'Proyecto Serenity Demo SCREEMPLAY'
                                         ])
                                        echo 'Reporte Html realizado con exito'
                                        currentBuild.result='SUCCESS'
                                    }
                                }
                    }
        }

        stage('Notificar') {
            steps {
                script {
                    if (currentBuild.result == 'UNSTABLE')
                        currentBuild.result = 'FAILURE'

                    if (currentBuild.result == 'SUCCESS')
                        emailext(
                                subject: "SERENITY DEMO- EJECUCION EXITOSA ESCENARIOS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                                body: """<p><b style="color:MediumSeaGreen;">EJECUCION EXITOSA:</b> Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
            				<p><b>Para verificar el estado de la ejecucion ingrese a:</b> &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                                to: "${CORREOS}"
                        )
                    if (currentBuild.result == 'FAILURE')
                        emailext(
                                subject: "SERENITY DEMO - EJECUCION FALLIDA ESCENARIOS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                                body: """<p><b style="color:red;">EJECUCION FALLIDA:</b> Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
            				<p><b>Para verificar el estado de la ejecucion ingrese a:</b> &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                                to: "${CORREOS}"
                        )
                }
            }
        }
    }
}