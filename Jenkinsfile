pipeline {
	agent any
	environment{
		PROJECT_TITLE = "NAME OF PROJECT"
		REPO_URL = "https://github.com/astemes/NAME_OF_REPOSITORY"
		AUTHOR = "Anton Sundqvist"
		INITIAL_RELEASE = 2022
		LV_PROJECT_PATH = "source\\Project.lvproj"
		LV_BUILD_SPEC = "Build Specification"
		LV_PACKAGE_BUILD_SPEC = "Package Build Specification"
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
				setLVBuildSpecVersion "${LV_PROJECT_PATH}", "${LV_BUILD_SPEC}", COMMIT_TAG
				buildLVBuildSpec "${LV_PROJECT_PATH}", "${LV_BUILD_SPEC}"
				
				//Build mkdocs documentation
				buildDocs "${PROJECT_TITLE}", "${REPO_URL}", "${AUTHOR}", "${INITIAL_RELEASE}"
				
				//If buildinga package for release, set the FILE_PATH variable
				setLVBuildSpecVersion "${LV_PROJECT_PATH}", "${LV_PACKAGE_BUILD_SPEC}", COMMIT_TAG
				script{FILE_PATH = buildLVBuildSpec "${LV_PROJECT_PATH}", "${LV_PACKAGE_BUILD_SPEC}"}
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