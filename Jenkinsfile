pipeline{
  agent {
    kubernetes {
      defaultContainer 'kaniko'
        yaml '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: gradle
        image: gradle:6.3-jdk14
        command:
        - sleep
        args:
        - 99d
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt        
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt
        - name: kaniko-secret
          mountPath: /kaniko/.docker
      restartPolicy: Never
      volumes:
      - name: shared-storage
        persistentVolumeClaim:
          claimName: jenkins-pv-claim
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
        '''
    }
  }
  triggers{
    pollSCM('*****')
  }
  stages {
    stage("Build the gradle project"){
      steps {
        sh "pwd"
        sh "chmod +x gradlew"
        sh "./gradlew build" 
        sh "mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt"
        }
      }
    }
    post {
      success{
       sh "echo 'FROM openjdk:8-jre' > Dockerfile" 
       sh "echo 'COPY ./calculator-0.01-SNAPSHOT.jar app.jar' >> Dockerfile"
       sh "echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile"
       sh "mv /mnt/calculator-0.0.1-SNAPSHOT.jar . /kaniko/executor --context 'pwd' --destination acoltrane/calculator-kankio:1.0"
      }
    }
  }
}




