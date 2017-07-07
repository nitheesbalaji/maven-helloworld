stage 'Compile'
node('localhost') {
    checkout scm
    // use for non multibranch: git 'https://github.com/amuniz/maven-helloworld.git'
    def mvnHome = tool 'maven-3'
    sh "${mvnHome}/bin/mvn clean install -DskipTests"
    stash 'working-copy'
}

stage 'Test'
parallel one: {
    node('localhost') {
        unstash 'working-copy'
        def mvnHome = tool 'maven-3'
        sh "${mvnHome}/bin/mvn test -Diterations=10"
    }
}, two: {
    node('localhost') {
        unstash 'working-copy'
        def mvnHome = tool 'maven-3'
        sh "${mvnHome}/bin/mvn test -Diterations=5"
    }
}, failFast: true

stage 'Code Quality'
node('localhost') {
    unstash 'working-copy'
    step([$class: 'CheckStylePublisher'])
    step([$class: 'FindBugsPublisher'])
    step([$class: 'PmdPublisher'])
}

stage name: 'Deploy', concurrency: 1
def path = input message: 'Where should I deploy this build?', parameters: [[$class: 'StringParameterDefinition', name: 'FILE_PATH']]
node('any') {
    unstash 'working-copy'
    sh "cp target/example-1.0-SNAPSHOT.jar ${path}"
}
