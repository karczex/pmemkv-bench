// SPDX-License-Identifier: BSD-3-Clause
/* Copyright 2019-2020, Intel Corporation */

/* Jenkinsfile - Scripts to create pmemkv and pmemkv_matrix jobs - to run with initial jenkins job. */

/* define common properties of pipelines: */
description_param = '''
	string {
		name('DESCRIPTION')
		defaultValue('')
		trim(true)
		description('Optional description for this execution')
	}
'''

label_param = '''
	string {
		name('LABEL')
		defaultValue('fedora')
		trim(true)
		description("Name of the node or group to run job on: rhel8_2, openSUSE15_2, fedora32, ubuntu20_04, ubuntu19.10, debian10, etc.")
	}
'''

repo_url_param = '''
	string {
		name('REPO_URL')
		defaultValue('https://github.com/pmem/pmemkv.git')
		trim(true)
		description("Git repository address")
	}
'''

branch_param = '''
	string {
		name('BRANCH')
		defaultValue('master')
		trim(true)
		description("Repository's branch")
	}
'''

test_options_param = '''
	string {
		name('TEST_OPTIONS')
		defaultValue('bench_scenarios/basic.json')
		trim(true)
		description("Test builds, as defined in run-build.sh")
	}
'''

test_additional_env = '''
	string {
		name('TEST_ADDITIONAL_ENV')
		defaultValue('')
		trim(true)
		description("Set environment variables for test execution")
	}
'''

type_param = '''
	choiceParam('TEST_TYPE', ['benchmarks'], 'Type of tests')
'''

coverage_param = '''
	choiceParam('COVERAGE', ['no', 'yes'], 'Enable code coverage')
'''

test_device = '''
	choiceParam('DEVICE_TYPE', ['PMEM', 'DAX', 'NONE'], 'Select tested device type. For PMEM and DAX, capable persistent memory must be present on the server.')
'''

gitlab_connection = '''
	properties {
		gitLabConnection {
			gitLabConnection('gitlabIT')
		}
	}
'''

email_options_param = '''
	string {
		name('EMAIL_RECIPIENTS')
		defaultValue('')
		trim(true)
		description("Recipients of the e-mail sent after execution, separated by the comma.")
	}
	booleanParam('SEND_RESULTS', true, 'Uncheck to disable sending email with report after execution')
'''

/* Branch with Jenkins libraries to pull: */
jenkins_files_branch_source = 'master'

/* Loads pipeline scripts from repository */
def remote_definition(current_job_script_path) {
    return """
		definition {
			cpsScm {
				scm {
					git {
						remote {
							url('https://github.com/pmem/pmemkv-tools.git')
							branch('${jenkins_files_branch_source}')
							scriptPath('${current_job_script_path}')
						}
					}
				}
			}
		}
	"""
}

environmental_variables = """
	environmentVariables {
		envs(
			api_lib: "dev-utils-kit/jenkins/lib/api.groovy",
			lib_path: "dev-utils-kit/jenkins/lib/",
			scripts_path: "dev-utils-kit/jenkins/scripts/",
			jenkins_files_repo: "https://github.com/pmem/pmemkv-tools.git",
			jenkins_files_branch: "*/${jenkins_files_branch_source}",
			info_addr: "https://pmem-val-jenkins.pact.intel.com/"
		)
	}
"""

triggers = '''
	triggers {
		gitlabPush {
			buildOnMergeRequestEvents(true)
			buildOnPushEvents(true)
			enableCiSkip(false)
			setBuildDescription(false)
			rebuildOpenMergeRequest('never')
		}
	}
'''

node {
	stage('pmemkv-tools'){
        jobDsl scriptText: """pipelineJob("pmemkv-tools") {
			parameters {
				${label_param}
				${branch_param}
				${type_param}
				${test_options_param}
				${test_additional_env}
				${test_device}
				${repo_url_param}
				${description_param}
				${coverage_param}
				${email_options_param}
			}
			${gitlab_connection}
			${environmental_variables}
			${triggers}
			${remote_definition 'pmemkv-tools.jenkins'}
		}"""
    }
    stage('pmemkv-tools_matrix'){
        jobDsl scriptText: """matrixJob("pmemkv-tools_matrix") {
        	parameters {
				matrixCombinationsParam('COMBINATIONS', '', 'choose which combinations to run')
				${branch_param}
				${coverage_param}
				${repo_url_param}
				${test_options_param}
				${test_additional_env}
				${test_device}
				${description_param}
				${email_options_param}
        	}
			axes {
		        text('DISTRO', 'ubuntu19_10', 'ubuntu20_04', 'fedora31')
		        text('TYPE', 'normal', 'building', 'compatibility')
		        label('master', 'master')
		    }
		    steps {
		        downstreamParameterized {
					trigger("pmemkv-tools") {
						parameters {
							predefinedProp('COV', '\${COVERAGE}')
							predefinedProp('TEST_TYPE', '\${TYPE}')
							predefinedProp('LABEL', '\${DISTRO}')
							predefinedProp('BRANCH', '\${BRANCH}')
							predefinedProp('TEST_OPTIONS', '\${TEST_OPTIONS}')
							predefinedProp('DEVICE_TYPE', '\${DEVICE_TYPE}')
							predefinedProp('REPO_URL', '\${REPO_URL}')
							predefinedProp('EMAIL_RECIPIENTS', '\${EMAIL_RECIPIENTS}')
							predefinedProp('SEND_EMAIL', '\${SEND_RESULTS}')
                            predefinedProp('DESCRIPTION', '\${DESCRIPTION}  #Triggered by pmemkv matrixJob #\${BUILD_NUMBER}   ->   \${JENKINS_URL}view/all/job/pmemkv-tools_matrix/\${BUILD_NUMBER}')
						}
					block {
						buildStepFailure('FAILURE')
						failure('FAILURE')
						unstable('UNSTABLE')
						}
					}
				}
		    }
		}"""
    }
}
