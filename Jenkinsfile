#!/usr/bin/env groovy

properties([
    buildDiscarder(
        logRotator(
            artifactDaysToKeepStr: '',
            artifactNumToKeepStr: '',
            daysToKeepStr: '',
            numToKeepStr: '50'
            )),
    pipelineTriggers([
        githubPush()
    ]),
    disableConcurrentBuilds()
])

node ('inbound4.3'){
    def error = null
    try {
        checkout scm
        def conf = readYaml(file: 'Jenkinsfile.yaml')
        currentBuild.displayName = "PIPELINE- " + conf.dockerImageName + ' rama ' + env.BRANCH_NAME
        currentBuild.description = "Despliegue del componente " + conf.dockerImageName + ' en la rama ' + env.BRANCH_NAME
        startJob(
            'Pipelines/credito-credicore-microservicios',
            [
                string(name: 'repo', value: conf.repository), 
                string(name: 'branch', value: env.BRANCH_NAME),
                string(name: 'credentials', value: conf.credentials),
                string(name: 'buildNumber', value: env.BUILD_NUMBER)
            ]
        )
    }    
    catch(caughtError) {
        error = caughtError
        currentBuild.result = 'FAILURE'
    }

    finally {
        notifyBuild(currentBuild.result)
        cleanWs()
        if (error) {
            throw error
        }
    }
}

def notifyBuild(String buildStatus = 'STARTED') {
    stage('NotifyBuild') {
        withAWSParameterStore(credentialsId: 'leeroy-jenkins', naming: 'basename', path: '/gb-corp-prod/jenkins/github/groups/', recursive: false, , regionName: 'us-east-1') {
        buildStatus = buildStatus ?: 'SUCCESS'
        def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
        def message =  """Cordial saludo equipo Credicore,
        
        El JOB: '${env.JOB_NAME} 
        Ejecución número: [${env.BUILD_NUMBER}]'
        Se ejecuto y su resultado es: ${buildStatus}
        Adjunto encontrara el log de ejecución, también puede encontrar más detalles en '${env.BUILD_URL}'
        
        Recuerde cualquier duda o novedad puede contactarse al correo devopsbancopopular@bancopopular.com.co"""
        emailext (
            attachLog: true, 
            subject: subject,
            to: "$env.CREDITO_CREDICORE",
            body: message,
        )
        }
    }
}  

def startJob (jobname, parameters){
    stage('StartJob'){
        build job: jobname, parameters: parameters
    }
}
