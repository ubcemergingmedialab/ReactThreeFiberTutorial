pipeline {
    agent { label 'linux' }
    tools {
        nodejs '24'
    }

    environment {
        // Create React App 4 uses webpack 4, whose MD4 hashing needs the legacy OpenSSL provider on Node 17+
        NODE_OPTIONS = '--openssl-legacy-provider'
        COREPACK_ENABLE_DOWNLOAD_PROMPT = '0'
    }

    stages {
        stage("Checkout"){
            steps {
                checkout scm
            }
        }
        stage("Install dependencies"){
            steps {
                // Yarn is not on the agents, so get Yarn 1 through Corepack into the workspace
                sh '''
                    mkdir -p "$WORKSPACE/.corepack"
                    corepack enable --install-directory "$WORKSPACE/.corepack"
                    export PATH="$WORKSPACE/.corepack:$PATH"
                    yarn install --frozen-lockfile
                '''
            }
        }
        stage("Build"){
            steps {
                sh '''
                    export PATH="$WORKSPACE/.corepack:$PATH"
                    yarn build
                '''
            }
        }
        stage("Archive"){
            steps {
                // Fingerprinting allows us to find where a build came from (ie archive files -> version)
                // allowEmptyArchive: false fails the build if no files match
                archiveArtifacts artifacts: 'build/**/*',
                                 fingerprint: true,
                                 allowEmptyArchive: false
            }
        }
    }
}
