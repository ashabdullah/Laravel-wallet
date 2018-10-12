node {
    
    stage('repository_pull') {
      // Checkout the master branch of the Laravel framework repository
      sh 'env | sort'
      git branch: 'master', url: 'https://MY_GIT_URL.git'
    }
    
    stage("composer_install") {
      // Run `composer update` as a shell script
      sh 'composer install'
    }
    
    stage("phpunit") {
      // Run PHPUnit
      sh 'vendor/bin/phpunit'

      // If this is the master or develop branch being built then run
      // additional integration tests
      if (["master", "develop"].contains(env.NODE_NAME)) {
          stage("integration_tests") {
              // Execute some scripts here
              // sh ''
          }
      }
      
    }
    
    stage("zip_archive") {
      // Zip the archive to prepare it for an S3 upload
      sh "zip -r ${env.BUILD_TAG}.zip ."
    }
    
    stage("s3_upload") {
      // Upload deployment assets to S3
      sh "aws s3 cp ${env.BUILD_TAG}.zip s3://MY_S3_BUCKET/ --region ap-southeast-2"
    }

    stage("codedeploy_execute") {
      // Deploy assets
      switch (env.NODE_NAME) {
        case "master":
          stage("codedeploy") {
            // Push zip to CodeDeploy
            sh "aws deploy push --application-name MY_SAMPLE_APP --s3-location s3://MY_S3_BUCKET/${env.BUILD_TAG}.zip --region ap-southeast-2 --source ./"
            // Deploy zip
            sh "aws deploy create-deployment --application-name MY_SAMPLE_APP --s3-location bucket=MY_S3_BUCKET,key=${env.BUILD_TAG}.zip,bundleType=zip --deployment-group-name MY_AWS_SERVER_ASG --deployment-config-name CodeDeployDefault.OneAtATime --description 'Deploying App Build: ${env.BUILD_NUMBER}' --region ap-southeast-2"
            }
        break
        case "develop":
          stage("codedeploy") {
            // Deploy to the test server
            }
        break
        default:
        // No deployments for other branches
        break
      }
    }

    stage('cleanup') {
    // Recursively delete all files and folders in the workspace
    // using the built-in pipeline command
    deleteDir()
    }
}
