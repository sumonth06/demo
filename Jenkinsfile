pipeline {
  agent {label 'IAC-Jenkins-Node'}
  
  environment {
    aws_accountnumber = "${aws_accountnumber}"
    environment_name  = "${environment_name}"
    cidr_block        = "${cidr_block}"
  }

  stages {
    stage("creating new env files"){
      steps{
        script{
          echo "script started  ......................."
          checkout([$class: 'GitSCM', branches: [[name: "${Branch}"]], userRemoteConfigs: [[credentialsId: 'your-credential-id', url: 'git-repo-url']]])

          sh "git pull"
          def sout= sh (
              // script: "find . -type d -name '${environment_name}s' | find . -type d -name '${aws_region}'",
              script: "find . -type d -name '${environment_name}'",
              returnStdout: true
          ).trim()
          echo "${sout}"
          if (sout.isEmpty()){
            echo "there is no folder with ${environment_name}"
            sh "cp -r ${copy_from_environment_name} ${environment_name}"
            sh "find ./${environment_name}/* -type f -exec sed -i 's/${copy_from_environment_name}/${environment_name}/g' {} \\;"
            sh "find ./${environment_name}/* -type f -exec sed -i 's/${copy_from_cidr_block}/${cidr_block}/g' {} \\;"
            sh "find ./${environment_name}/* -type f -exec sed -i 's/${copy_from_aws_accountnumber}/${aws_accountnumber}/g' {} \\;"
            sh "find ./${environment_name}/* -type f -exec sed -i 's/${copy_from_mongo_region}/${mongo_region}/g' {} \\;"
            sh "find ./${environment_name}/* -type f -exec sed -i 's/${copy_from_peering_region}/${peering_region}/g' {} \\;"
            sh "find ./${environment_name}/* -type f -exec sed -i 's/${copy_from_aws_region}/${aws_region}/g' {} \\;"
          }
          else{
            error("there is already a folder with ${environment_name}")
          }
          
          // sh "git add ."
          // retry(2){
          //   sh "sleep \$(shuf -i 10-30 -n 1)"
          //   sh "git pull"
          //   sh "git branch"
          //   sh "git diff-index --quiet ${Branch} || git commit -m 'Jenkins Pipeline commit to add & update new environment ${environment_name} details 1.account number 2.environment name 3.cidr_bock' && git push"
          // } 
        }                   
      }
    }
   input message: 'Commit files?', ok: 'Proceed', submitter: 'user', parameters: [booleanParam(defaultValue: false, description: 'Commit files?', name: 'commit')]

    stage('S3 bucket'){
      steps{
        script{
          if(s3_buckets){
            def texts = s3_buckets.split(',')
            // sh '''
            //   source ./init_s3.sh ${aws_accountnumber} ${rolename} &>outputfile.out
            // '''
            
}

            for (s3_txt in texts) {
              echo "creating s3 bucket ${s3_txt}"
              withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-credentials-id', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
    sh"source ./init_s3.sh ${aws_accountnumber} ${rolename} ${s3_txt} ${aws_region} &>outputfile.out"
}

              // sh"aws s3api create-bucket --bucket ${s3_txt} --region ${region} --create-bucket-configuration LocationConstraint=${region}"
              // '''
            }
          }
          else{
            echo "proceeding with the build without any s3 bucket creation"
          }
        }
      }
    }
    stage("commit files"){
      steps{
        script{
          echo "script started  ......................."
          sh "git checkout ${Branch}"
          sh "git pull"
          sh "git add -- . :!outputfile.out :!outputFile"
          retry(2){
            sh "sleep \$(shuf -i 10-30 -n 1)"
            sh "git pull"
            sh "git branch"
            sh "git diff-index --quiet ${Branch} || git commit -m 'Jenkins Pipeline commit to add & update new environment ${environment_name} details 1.account number 2.environment name 3.cidr_bock' && git push"
          } 
        }                   
      }
    }
  }
}
