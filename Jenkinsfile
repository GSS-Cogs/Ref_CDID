pipeline {
    agent {
        label 'master'
    }
    stages {
        stage('Create missing CDIDs') {
            agent {
                docker {
                    image 'cloudfluff/databaker'
                    reuseNode true
                }
            }
            steps {
                sh 'jupyter-nbconvert --to python --stdout cdid2rdf.ipynb | python'
            }
        }
        stage('Upload CDIDs') {
            steps {
                script {
                    def pmd = pmdConfig("pmd")
                    String draftId
                    try {
                        oldDraftId = pmd.drafter.findDraftset(env.JOB_NAME).id
                    } catch (Exception e) {
                        echo "No old draftset."
                    } finally {
                        draftId = pmd.drafter.createDraftset(env.JOB_NAME).id
                    }
                    String graph = "http://gss-data.org.uk/def/cdid_legacy"
                    pmd.drafter.deleteGraph(draftId, graph)
                    pmd.drafter.addData(draftId, "${WORKSPACE}/out/cdids.ttl", 'text/turtle', 'UTF-8', graph)
                }
            }
        }
        stage('Publish') {
            steps {
                script {
                    jobDraft.publish()
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts 'out/*'
        }
    }
}
