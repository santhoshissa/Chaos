import groovy.json.JsonSlurperClassic
def output;
@NonCPS
def parseJsonToMap(String json) {
    final slurper = new JsonSlurperClassic()
    return new HashMap<>(slurper.parseText(json))
}
pipeline {
    agent any
    stages {
        stage('TRIGGER_JMETER_TEST') {
            steps {
                    script {
                         //sh "/root/RajaAnbazhagan/Applications/apache-jmeter-5.5/bin/jmeter -n -t /root/RajaAnbazhagan/Applications/Docker_JMeter/docker-jmeter/NGINX_Script.jmx -JDuration=1800 -JThreads=1 &"
                         sleep 120
                    }
                }
        }
        
                stage('TRIGGER_KEPTN_EVALUATION_PRE_CHAOS') {
            steps {
                    script {
                        TRIGGER_KEPTN = sh (
                        script: 'keptn trigger evaluation --project=sampleapp --stage=perf --service=nginx --timeframe=5m',
                        returnStdout: true
                        ).trim()
                        x = TRIGGER_KEPTN.size()
                        z = TRIGGER_KEPTN.indexOf(":")
                        env.keptnid = TRIGGER_KEPTN.substring(z+2,x)
                        println "keptn id-->"
                        println keptnid
                        //sleep 60
                        }
                }
        }
                            stage('GET_KEPTN_RESULT_PRE_CHAOS') {
            steps {
                    script {
                         println "----------"
                         println keptnid
                         println "----------"
                        KEPTN_EVAL = sh (
                        script: 'keptn get event evaluation.finished --keptn-context=$keptnid',
                        returnStdout: true
                        ).trim()
                        x = KEPTN_EVAL.size()
                        z = KEPTN_EVAL.indexOf("{")
                        y = KEPTN_EVAL.substring(z,x)
                        def map = parseJsonToMap(y)
                        echo  "KEPTN EVALUATION = ${map.data.result}"
                        def result = map.data.result
                        //def pjson = new groovy.json.JsonSlurper().parseText("${GIT_COMMIT_EMAIL}")
                        if(result.contains("Pass"))
                        {
                            echo "Pass"
                        }
                        else if(result.contains("warning"))
                        {
                            //echo "warning"
                            //currentBuild.result = 'ABORTED';
                            return;
                        }
                        else
                        {
                            echo "Fail"
                            currentBuild.result = 'ABORTED';
                            return;
                        }
                }
        }
                            }
stage('EXECUTE_CHAOS') {
            steps {
                script {
                CREATE_CHAOS = sh (
                    script: 'kubectl apply -f ${Experiment}_param.yaml',
                        //script: '',
                        returnStdout: true
                        ).trim()
                echo "${CREATE_CHAOS}"
                sleep 180
                }
            }
        }
        stage('GET_CHAOS_RESULT') {
            steps {
                script {
                GET_CHAOS_RS = sh (
                        script: 'kubectl describe chaosresult engine-nginx-pod-network-latency -n sampleapp',
                        returnStdout: true
                        ).trim()
                echo "${GET_CHAOS_RS}"
                start = GET_CHAOS_RS.indexOf("Verdict:")
                end = GET_CHAOS_RS.indexOf("History")
                y = GET_CHAOS_RS.substring(start+8,end)
                echo " CHAOS RESULT -- ${y}"
                }
            }
        }
stage('TRIGGER_KEPTN_EVALUATION_POST_CHAOS') {
            steps {
                    script {
                        TRIGGER_EVAL = sh (
                        script: 'keptn trigger evaluation --project=sampleapp --stage=perf --service=nginx --timeframe=5m',
                        returnStdout: true
                        ).trim()
                        echo "${TRIGGER_EVAL}"
                        x = TRIGGER_EVAL.size()
                        z = TRIGGER_EVAL.indexOf(":")
                        env.keptnid = TRIGGER_EVAL.substring(z+2,x)
                        println "keptn id-->"
                        println keptnid
                        sleep 60
                        }
                }
        }
                            stage('GET_KEPTN_RESULT_POST_CHAOS_') {
            steps {
                    script {
                         println "----------"
                         println keptnid
                         println "----------"
                        KEPTN_RS = sh (
                        script: 'keptn get event evaluation.finished --keptn-context=$keptnid',
                        returnStdout: true
                        ).trim()
                        x = KEPTN_RS.size()
                        z = KEPTN_RS.indexOf("{")
                        y = KEPTN_RS.substring(z,x)
                        def map = parseJsonToMap(y)
                        echo  "KEPTN RESULT = ${map.data.result}"
                        }
                }
        }
    }
}
