pipeline {
	agent {
        label 'LV2020'
    }
	environment{
		PROJECT_TITLE = "LUnit Parameterized Test Add-on"
		REPO_URL = "https://github.com/astemes/astemes-lunit-parameterized-test"
		AUTHOR = "Anton Sundqvist"
		INITIAL_RELEASE = 2024
		LV_PROJECT_PATH = "source\\Parameterized Test.lvproj"
		LV_BUILD_SPEC = "LUnit Parameterized Test Case"
		LV_VIPB_PATH = "source\\LUnit.vipb"
		LV_VERSION = "20.0"
	}
	stages {
		stage('Initialize') {
			steps {
				library 'astemes-build-support'
				script{COMMIT_TAG = gitTag()}
				initWorkspace()
				dir("build_support"){
					pullBuildSupport()
					initPythonVenv "requirements.txt"
				}
				pullSubmodules()
			}
		}
		stage('Test') {
			steps {
				runLUnit "${LV_PROJECT_PATH}"
				// Empty results set to true for the template to pass build
				junit testResults: "reports\\*.xml", allowEmptyResults: true
			}
		}
		stage('Build') {
			steps {
				//Execute LabVIEW build spec
				buildLVBuildSpec "${LV_PROJECT_PATH}", "${LV_BUILD_SPEC}"
				//Build mkdocs documentation
				buildDocs "${PROJECT_TITLE}", "${REPO_URL}", "${AUTHOR}", "${INITIAL_RELEASE}"
			}
		}
		stage('Deploy') {
			when{
				expression{
					!COMMIT_TAG.isEmpty()
				}
			}
			environment{
				GITHUB_TOKEN = credentials('github-token')
			}
			steps{
				deployGithubPages()
				deployGithubRelease "${REPO_URL}", "${COMMIT_TAG}", "${FILE_PATH}"
				script{VIP_FILE_PATH = buildVIPackage "${LV_VIPB_PATH}", "${LV_VERSION}", "${COMMIT_TAG}"}
				deployGithubPages()
				deployGithubRelease "${REPO_URL}", "${COMMIT_TAG}", "${VIP_FILE_PATH}"
			}
		}
	}	
	post{
		regression{
			sendMail "anton.sundqvist@astemes.com"
		}
	}
	options {
		buildDiscarder(logRotator(daysToKeepStr: '3', numToKeepStr: '5'))
	}
}