def awesomeVersion = [:] //making map to store custom data with their key 
//def awesomeVersion = 'UNKNOWN'
def endtime = [:]

pipeline {
    agent any
    stages {
        stage('Test-Start Time Capture') {
            steps {
                echo 'Capturing last Build Number and ingesting Data to Influx-DB'

                script {
                    //def buildNumber = Jenkins.instance.getItem('jobName').lastSuccessfulBuild.number // Use This if you want to get build.no. of successfull build only
                    def buildNumber = Jenkins.instance.getItem('TEST').lastBuild.number
                    def newbuildNumber = buildNumber.next() //If above variable is picking second latest build no. then we will use this variable to pick latest   
                        echo "${buildNumber}"
                        echo "${newbuildNumber}"
                        sh "cat $JENKINS_HOME/jobs/TEST/builds/${buildNumber}/log|grep -iw 'Test execution/login started at:'|tail -n 1|cut -d ':' -f2,3,4|sed 's/^ *//g'|tr ' ' '_'> $JENKINS_HOME/userContent/monitoring/cap.txt"
                        awesomeVersion['yo'] = sh(returnStdout: true, script: 'cat $JENKINS_HOME/userContent/monitoring/cap.txt').trim()
                        //awesomeVersion = sh(returnStdout: true, script: 'cat $JENKINS_HOME/userContent/monitoring/cap.txt').trim()
                }
               // sh "cat $JENKINS_HOME/userContent/monitoring/cap.txt"
                echo  "${awesomeVersion}"
                 //influxDbPublisher customPrefix: '',customProjectName: 'Test-shows', customData:awesomeVersion,jenkinsEnvParameterField: '''environment=test''',selectedTarget:'Monitoring'
                 influxDbPublisher(selectedTarget: 'Monitoring', customProjectName: 'Test_show', customData: awesomeVersion)
                
            }
        }
        
        stage('Checking Build Result of Job Before getting END-TIME'){
            steps{
                echo 'Checking Last Build Status of TEST JOb'
                script{
                    def job = jenkins.model.Jenkins.instance.getItemByFullName("TEST")
                    def result = job.getLastBuild().getResult().toString()
                    env.result = result
                      }
                echo "The LAST BUILD STATUS is : ${result}"
                }
          
            
        }
        
        stage('End Time Capture'){
            steps{
                echo 'Capturing END TIME OF TEST-CASE'
                
                script{
                    if (result == 'SUCCESS'){
                    def buildNumber = Jenkins.instance.getItem('TEST').lastBuild.number
                    sh "cat $JENKINS_HOME/jobs/TEST/builds/${buildNumber}/log|grep -iw 'Test execution completed at:'|tail -n 1|cut -d ':' -f2,3,4|sed 's/^ *//g'|tr ' ' '_'> $JENKINS_HOME/userContent/monitoring/end1.txt"
                    endtime['end_t'] = sh(returnStdout: true, script: 'cat $JENKINS_HOME/userContent/monitoring/end1.txt').trim()
                    echo "${endtime}"
                    influxDbPublisher(selectedTarget: 'Monitoring', customProjectName: 'Test_show', customData: endtime)
                    }
                    //else (result == 'FAILURE' || result == 'UNSTABLE' || result == 'ABORTED') {
                    else {    
                        echo "TEST RUN and the STATUS is ${result}"
                        endtime['end_t'] = 'NOT_SUCCESS-1' 
                        influxDbPublisher(selectedTarget: 'Monitoring', customProjectName: 'Test_show', customData: endtime)
                    }
                }
                  
                    
                }
            }
        
    }
}

