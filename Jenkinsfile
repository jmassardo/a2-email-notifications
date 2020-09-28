pipeline {
  agent any
  stages {
    stage('Output Test') {
      parallel {
        stage('Shell Output') {
          steps {
            sh '''
            echo "$node_name experienced a $type."
            echo "$total_number_of_passed_tests / $total_number_of_failed_tests / $total_number_of_skipped_tests / $total_number_of_tests"
          '''
          }
        }

        stage('Email Extension') {
          steps {
            emailext(subject: '$type for $node_name', body: '$total_number_of_passed_tests / $total_number_of_failed_tests / $total_number_of_skipped_tests / $total_number_of_tests')
          }
        }

      }
    }

  }
  triggers {
    GenericTrigger(genericVariables: [
                  [key: 'type', value: '$.type'],
                  [key: 'total_number_of_tests', value: '$.total_number_of_tests'],
                  [key: 'total_number_of_skipped_tests', value: '$.total_number_of_skipped_tests'],
                  [key: 'total_number_of_passed_tests', value: '$.total_number_of_passed_tests'],
                  [key: 'total_number_of_failed_tests', value: '$.total_number_of_failed_tests'],
                  [key: 'number_of_failed_critical_tests', value: '$.number_of_failed_critical_tests'],
                  [key: 'number_of_critical_tests', value: '$.number_of_critical_tests'],
                  [key: 'node_name', value: '$.node_name'],
                  [key: 'automate_failure_url', value: '$.automate_failure_url']
                 ], causeString: '$type on $node_name', regexpFilterExpression: '', regexpFilterText: '', printContributedVariables: true, printPostContent: true, token: '1')
    }
  }