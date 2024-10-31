pipeline {
	agent {
        label 'LV2020'
    }
	environment{
		PROJECT_TITLE = "LUnit Parameterized Test Add-on"
		REPO_URL = "https://github.com/Astemes/astemes-lunit-parameterized-test"
		AUTHOR = "Anton Sundqvist"
		INITIAL_RELEASE = 2024
		LV_PROJECT_PATH = "source\\Parameterized Test.lvproj"
		LV_BUILD_SPEC = "LUnit Parameterized Test Case"
		LV_VIPB_PATH = "LUnit Parameterized Add-On.vipb"
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
				script{VIP_FILE_PATH = buildVIPackage "${LV_VIPB_PATH}", "${LV_VERSION}", "${COMMIT_TAG}"}
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