pipeline {
  agent any

    tools {
      nodejs "Node20"
    }

  environment {
    NOMBRE = "ALAN"
  }

  stages {
    stage('Clonar repositorio con codigo'){
      steps{
        dir('seminario'){
          git "https://github.com/alanfvn/seminario"
        }
      }
    }

    stage('Limpiar datos de Docker'){
      steps{
        dir('seminario'){
          sh 'docker system prune -a --volumes -f'
        }
      }
    }

    stage('Iniciar contenedor'){
      steps{
        dir('seminario'){
          sh 'echo "DEBUG=FALSE" > .env'
          sh 'if [ -e db.sqlite3 ]; then rm db.sqlite3; fi'
          sh 'touch db.sqlite3'
          sh 'chmod 777 db.sqlite3'
          sh 'docker compose up -d --wait'
          sh 'docker compose ps'
        }
      }
    }

    stage('Hacer init al setup de Django'){
      steps{
        dir('seminario'){
            sh 'docker compose exec web python manage.py collectstatic'
            sh 'docker compose exec web python manage.py migrate'
            sh 'docker compose exec web python manage.py create_test_items'
        }
      }
    }
    
    stage('Clonar pruebas'){
      steps{
        dir('pruebas'){
          git "https://github.com/alanfvn/qa-proyecto"
        }
      }
    }

    stage('Instalar dependencias de las pruebas'){
      steps{
        dir('pruebas'){
          sh 'npm i'
        }
      }
    }

    stage('Ejecutar pruebas'){
      steps{
        dir('pruebas'){
          sh 'NO_COLOR=1 npx cypress run'
        }
      }
    }

    stage('Crear reporte'){
      steps{
        dir('pruebas'){
          npx mochawesome-merge "cypress/results/*.json" > mochawesome.json
          npx marge mochawesome.json
        }
      }
    }

  }

  post {
    always {
      dir('seminario'){
        sh 'docker compose down --remove-orphans -v'
        sh 'docker compose ps'
      }

      dir('pruebas'){
        script{
          def htmlContent = readFile 'mochawesome-report/mochawesome.html'
            emailext subject: '$DEFAULT_SUBJECT',
                     body: htmlContent,
                     recipientProviders: [
                       [$class: 'CulpritsRecipientProvider'],
                       [$class: 'DevelopersRecipientProvider'],
                       [$class: 'RequesterRecipientProvider']
                     ], 
                     replyTo: '$DEFAULT_REPLYTO',
                     to: '$DEFAULT_RECIPIENTS',
                     mimeType: 'text/html'
        }
      }
    }
  }
}
