//https://github.com/coresolutions-ltd/jenkins-terraform-pipeline

pipeline {
    agent any

    environment {
       // AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
       // AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
       // AWS_DEFAULT_REGION    = "eu-west-1"
       // BACKEND_BUCKET        = "jenkins-terraform-pipeline-state"
    }

    parameters {
        choice(
            name: 'Action',
            choices: ['Build', 'Destroy'],
            description: 'The action to take'
        )
        choice(
            name: 'Colour',
            choices: ['Blue', 'Green'],
            description: 'The environment to use'
        )
    }

    stages {
        stage('Init') {
            steps {
                terraformInit()
            }
        }
        stage('Plan') {
            steps {
                terraformPlan()
            }
        }
        stage('Approval') {
            steps {
                input(message: 'Apply Terraform ?')
            }
        }
        stage('Apply') {
            steps {
                terraformApply()
            }
        }
        stage('Validate') {
            steps {
                inspecValidation()
            }
        }
    }
    post {
        always {
            echo 'Deleting Directory!'
            deleteDir()
        }
    }
}

def terraformInit() {
    sh("""
        cd terraform;
        terraform init -backend-config="bucket=${env.BACKEND_BUCKET}" -backend-config="key=demo.tfstate"
        terraform workspace select ${params.Colour.toLowerCase()} || terraform workspace new ${params.Colour.toLowerCase()}
    """)
}

def terraformPlan() {
    // Setting Terraform Destroy flag
    if(params.Action == 'Destroy') {
        env.DESTROY = '-destroy'
    } else {
        env.DESTROY = ""
    }

    sh("""
        cd terraform;
        terraform plan ${env.DESTROY} -var-file=${params.Colour.toLowerCase()}.tfvars -no-color -out=tfout
    """)
}

def terraformApply() {
    sh("""
        cd terraform;
        terraform apply tfout -no-color
        //mkdir ../Inspec/files/
        //terraform output --json > ../../Inspec/files/output.json
    """)
}

def inspecValidation() {
    sh("""
        inspec exec Inspec/ -t aws:// --input workspace=${params.Colour}
    """)
}