def user = "iflowfor8hours"
def cfUser = "${cf_username}"
def cfPassword = "${cf_password}"
def cfOrg = "${cf_org}"
def sonarUrl = "${sonar_url}"
def cfSpace = "${cf_space}"
def domain = "${app_domain}"
def flow

stage 'compile'
node {
	git([url: "https://github.com/${user}/sample-spring-cloud-svc-ci.git", branch: 'master'])
	flow = load 'ci/pipeline.groovy'
	flow.build()
}

stage 'test'
node {
	git([url: "https://github.com/${user}/sample-spring-cloud-svc-ci.git", branch: 'master'])
	flow = load 'ci/pipeline.groovy'
	flow.clean_test()
}

stage 'publish-sonar-results'
node {
	flow.sonar(sonarUrl)
}

stage 'deploy-to-development'
node {
	git([url: "https://github.com/${user}/sample-spring-cloud-svc-ci.git", branch: 'master'])
	flow = load 'ci/pipeline.groovy'
	flow.push('api.local.pcfdev.io', "${cfUser}", "${cfPassword}", "${cfOrg}", "${cfSpace}", "${app_domain}", 'sample-spring-cloud-svc-ci-dev')
}

stage 'run-tests-on-dev'
parallel(
	smokeTests: {
		node {
			git([url: "https://github.com/${user}/sample-spring-cloud-svc-ci.git", branch: 'master'])
			flow = load 'ci/pipeline.groovy'
			flow.runSmokeTests('api.local.pcfdev.io', "${cfUser}", "${cfPassword}", "${cfOrg}", "${cfSpace}", "${app_domain}", 'sample-spring-cloud-svc-ci-dev')
		}
	},
	acceptanceTests: {
		node {
			git([url: "https://github.com/${user}/sample-spring-cloud-svc-ci.git", branch: 'master'])
			flow = load 'ci/pipeline.groovy'
			flow.runAcceptanceTests('api.local.pcfdev.io', "${cfUser}", "${cfPassword}", "${cfOrg}", "${cfSpace}", "${app_domain}", 'sample-spring-cloud-svc-ci-dev')
		}
	}
)

stage name: 'deploy-to-test', concurrency: 1
node {
	git([url: "https://github.com/${user}/sample-spring-cloud-svc-ci.git", branch: 'master'])
	flow = load 'ci/pipeline.groovy'
	flow.pushIf('api.local.pcfdev.io', "${cfUser}", "${cfPassword}", "${cfOrg}", 'test', "${app_domain}", 'sample-spring-cloud-svc-ci-test')
}

stage 'run-tests-on-test'
parallel(
	smokeTests: {
		node {
			git([url: "https://github.com/${user}/sample-spring-cloud-svc-ci.git", branch: 'master'])
			flow = load 'ci/pipeline.groovy'
			flow.runSmokeTests('api.local.pcfdev.io', "${cfUser}", "${cfPassword}", "${cfOrg}", 'test', "${app_domain}", 'sample-spring-cloud-svc-ci-test')
		}
	},
	acceptanceTests: {
		node {
			git([url: "https://github.com/${user}/sample-spring-cloud-svc-ci.git", branch: 'master'])
			flow = load 'ci/pipeline.groovy'
			flow.runAcceptanceTests('api.local.pcfdev.io', "${cfUser}", "${cfPassword}", "${cfOrg}", 'test', "${app_domain}", 'sample-spring-cloud-svc-ci-test')
		}
	}
)

input message: "Does https://sample-spring-cloud-svc-ci-prod.cfapps.io look good?"
try {
    checkpoint('Before production')
} catch (NoSuchMethodError _) {
    echo 'Checkpoint feature available in CloudBees Jenkins Enterprise.'
}

stage name: 'deploy-to-prod', concurrency: 1
node {
	git([url: "https://github.com/${user}/sample-spring-cloud-svc-ci.git", branch: 'master'])
	flow = load 'ci/pipeline.groovy'
	flow.pushIf('api.local.pcfdev.io', "${cfUser}", "${cfPassword}", "${cfOrg}", 'production', "${app_domain}", 'sample-spring-cloud-svc-ci-prod')
}
