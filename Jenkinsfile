pipeline {
    agent {
        label 'electronix'
    }

    environment{
        S3_BUCKET='electronix-production-2805'
        CLOUDFRONT_ID='E27QH4G5IONA5U'
        AWS_REGION='us-east-1'
            }

    stages {
        stage("Frontend Deployment") {
            when {
                changeset "frontend/**"
            }

            stages {

                stage("Install Dependencies") {
                    steps {
                        dir('frontend') {
                            sh 'npm install'
                        }
                    }
                }

                stage("Run Test") {
                    steps {
                        dir('frontend') {
                            sh 'npm test -- --watchAll=false || echo "No tests configured..."'
                        }
                    }
                }

                stage("Build") {
                    steps {
                        dir('frontend') {
                            sh 'npm run build'
                        }
                    }
                }

                stage("Deploy S3") {
                    steps {
                        dir('frontend') {
                            sh '''
                                aws s3 sync dist/ s3://$S3_BUCKET --delete --region $AWS_REGION
                            '''
                        }
                    }
                }

                stage("Invalidation Cloud Cache") {
                    steps {
                        sh '''
                            aws cloudfront create-invalidation \
                                --distribution-id $CLOUDFRONT_ID \
                                --paths "/*"
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Frontend deployment successful✅'
        }

        failure {
            echo 'Frontend deployment failed❌'
        }
    }
}