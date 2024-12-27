pipeline{
    agent any
    stages{
        stage ('Getcode'){
            steps{
                git 'https://github.com/jhdelca1/helloworld.git'
            }
        }
        stage ('Build'){
            steps{
                echo 'No hacemos nada aquí va python. Jenkinsfile'
                bat 'dir'
            }
        }
        stage('Test'){
            parallel{
                stage('Test Unit'){
                    steps{
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                        bat '''
                            set PYTHONPATH=%WORKSPACE%
                            python -m pytest --version
                            pytest --junitxml=result-unit.xml test\\unit
                        '''   
                        }
                    }
                }

                stage('RestWIREMOCK'){
                    steps{
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            bat '''                                
                                start java -jar C:\\Jose\\MasterDEVOPS\\Master\\Software\\wiremock-standalone-3.10.0.jar --port 9090 --root-dir  C:\\Jose\\MasterDEVOPS\\Practicasdeclase\\Practica1\\helloworld\\test\\wiremock
                                set PYTHONPATH=%WORKSPACE%                                
                            '''
                        }
                    }
                }
                stage('RestFLASK'){
                    steps{
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            bat '''
                                set FLASK_APP=app\\api.py
                                start flask run
                                ping -n 5 127.0.0.1
                            '''
                        }
                    }
                }
            }
           
        }
        stage('EjecutarRest'){
            steps{
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    bat '''
                        set PYTHONPATH=%WORKSPACE%
                        pytest --junitxml=result-rest.xml test\\rest
                    '''
                }
            }
        }
        stage('Results'){
            steps{
                junit 'result*.xml'
            }
        }
        
    }
}
