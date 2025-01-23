pipeline {
    agent any
    parameters {
        //He puesto un parametro para elegir la rama que descargo (master para el reto 1 y feature_fix_coverage para el reto 3
        choice choices: ['master', 'feature_fix_coverage'], description: 'Valores a elegir', name: 'Rama'
    }
    stages {
        stage('Get Code') {
            steps {
                // Obtener el codigo
        		git branch: '${rama}', url:'https://github.com/jhdelca1/helloworld.git'
        		bat 'dir'
        		echo WORKSPACE
            }
        }
		
	    stage('Unit') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    //En esta etapa ejecutamos pytest a la vez que coverage para solamente ejecutar una vez pytest
                    bat '''
                        set PYTHONPATH=%WORKSPACE%
                        coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest --junitxml=result-unit.xml test\\unit
                        coverage xml
                    '''
                    //Tratamos los resultados de pytest con junit
                    junit 'result*.xml'
                }
            }
        }

        stage('Static') {
    		steps {
    			//Ejecutamos las pruebas estaticas y ejecutamos el plugin para tratar los resultados
                bat '''
    				flake8 --exit-zero --format=pylint app >flake8.out
    			'''    
    			recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
    		    
	        }
	    }
        stage('Security') {
    		steps {
			    // Se pone exit-zero para que no de error al encontrar algun problema, y se pare la ejecucion
			    // Esta etapa puede dar error debido a que el plugin de Jenkins no lee correctamente el fichero de salida
    			bat '''
    				bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
    			'''
    	
    			recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
    		    
		    }
	    }
	    stage('Coverage') {
            steps {
                script {
                    //   Tratamos el fichero de cobertura generado en la etapa unit
			        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){ 
				        cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '90,0,80', lineCoverageTargets: '95,0,85'
			        }
			        
                    
                }
            }
        }
		
        stage('Performance') {
    		steps {
			    //Levantamos flask 
			    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    bat '''
                        set FLASK_APP=app\\api.py
                        start flask run
                        ping -n 5 127.0.0.1
                    '''
                }	

			    // Se pone exit-zero para que no de error al encontrar algun problema, y se pare la ejecucion
    			bat '''
    				//Se pone la rutadonde tengas el ejecutable de jmeter
				    C:\\Jose\\MasterDEVOPS\\Master\\Software\\jmeter\\apache-jmeter-5.6.3\\bin\\jmeter -n -t test\\jmeter\\flask.jmx -f -l flask.jtl
    			'''
    			perfReport sourceDataFiles: 'flask.jtl'    			
    		    
		}
	}
}

 	
}
