stage('test') {
    node ('dockerslave'){
        git "${GITREPO}"
        sh 'chmod a+x ./run_test.sh'
        sh './run_test.sh'
    }
}

node() {
    git '${GITREPO}'
    stage('build the image') {
        withDockerServer([credentialsId: 'dockerhost', uri: "tcp://${DOCKERHOST}:2376"]) {
            docker.build '${DOCKER_REGISTRY_USER}/rsvpapp:mooc'
        }
    }
    
    stage('push the image to DockerHub') {
        withDockerServer([credentialsId: 'dockerhost', uri: "tcp://${DOCKERHOST}:2376"]) {
            withDockerRegistry([credentialsId: 'yadkovdockerhub']) {
                docker.image("${DOCKER_REGISTRY_USER}/rsvpapp:mooc").push()
            }
        }
    }
    
    stage('deploy the image to staging server') {
        withDockerServer([credentialsId: 'staging', uri: "tcp://${STAGING}:2376"]){
            sh '/usr/local/bin/docker-compose pull'
            sh '/usr/local/bin/docker-compose -p rsvp_staging up -d'
        }
        input "Check application running at http://${STAGING}:5000 Looks good?"
        withDockerServer([credentialsId: 'staging', uri: "tcp://${STAGING}:2376"]) {
            sh '/usr/local/bin/docker-compose -p rsvp_staging down -v'
        }
    }
    
    stage('deploy in production'){
        withDockerServer([credentialsId: 'production', uri:"tcp://${PRODUCTION}:2376"]) {
            sh 'docker stack deploy -c docker-stack.yaml myrsvpapp'
        }
    }
}
