podTemplate(yaml: '''
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
''') {
  node(POD_LABEL) {
    stage('Build a gradle project') {
            container('gradle') {
            git url: 'https://github.com/austineisele/week7.git' 
        stage('Build a gradle project') {
          sh '''
          pwd
          chmod +x gradlew
          ./gradlew build
          mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
          '''
          //variable to test for execptions
          testPassed = true
          containerName = ""
          //first playground stage. Runs no tests. No container built
          stage("playground test"){
            if(env.BRANCH_NAME == 'playground') {
              echo "this is the playground branch. No tests have been run"
              testPassed = false
            }

          }
          //feature test does not test for CodeCoverage. Conatiner built
          // if successful
          stage("feature test"){
            if(env.BRANCH_NAME == 'feature'){
              echo "running tests on the feature branch"
                try{
                  sh '''
                    pwd
                    ./gradlew checkstyleMain
                    ./gradlew test
                    ./gradlew jacocoTestReport
                    '''
                    containerName = "calculator-feature-kaniko:0.1"
                }
              catch (Exception e){
                  testPassed = false
                  echo 'Error: ' + e.toString()
              }
            }

          }
          //main does all tests
          //container build if successful
          stage("main test"){
            if(env.BRANCH_NAME == 'main'){
              echo "running tests on the main branch"
                try{
                  sh '''
                    pwd
                    ./gradlew checkstyleMain
                    ./gradlew test
                    ./gradlew jacocoTestReport
                    ./jacocoTestCoverageVerification
                    ./jacocoTestReport
                    '''
                    containerName = "calculator-kaniko:1.0"
                }
              catch (Exception e){
                testPassed = false
                echo 'Error: ' + e.toString() 
              }
            }
          }
        }
      }
    }

    stage('Build Java Image') {
      container('kaniko') {
        stage('Build a gradle project') {
          if(testPassed == true){
          sh '''
            echo 'FROM openjdk:8-jre' > Dockerfile
            echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
            echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
            mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
            /kaniko/executor --context `pwd` --destination acoltrane/$containerName
          '''
          }
        }
      }
    }

  }
}
