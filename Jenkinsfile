@Library('jenkins-helpers@v0.1.8') _

podTemplate(
    label: 'jnlp-cognite-sdk-python',
    containers: [containerTemplate(name: 'python',
                                   image: 'python:3.6.4',
                                   command: '/bin/cat -',
                                   resourceRequestCpu: '1000m',
                                   resourceRequestMemory: '500Mi',
                                   resourceLimitCpu: '1000m',
                                   resourceLimitMemory: '500Mi',
                                   envVars: [envVar(key: 'PYTHONPATH', value: '/usr/local/bin')],
                                   ttyEnabled: true)],
    volumes: [secretVolume(secretName: 'jenkins-docker-builder',
                           mountPath: '/jenkins-docker-builder',
                           readOnly: true),
              secretVolume(secretName: 'pypi-credentials',
                           mountPath: '/pypi',
                           readOnly: true)]) {
    node('jnlp-cognite-sdk-python') {
        def gitCommit
        container('jnlp') {
            stage('Checkout') {
                checkout(scm)
                gitCommit = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            }
        }
        container('python') {
            stage('Install pipenv') {
                sh("pip3 install pipenv==9.0.1")
            }
            stage('Install dependencies') {
                sh("pipenv install")
                sh("pipenv run pip3 install .")
            }
            stage('Test') {
                sh("cd unit_tests && pipenv run python3 run_tests.py")
            }
            stage('Build') {
                sh("pipenv run python3 setup.py sdist")
                sh("pipenv run python3 setup.py bdist_wheel")
            }
            def pipVersion = sh(returnStdout: true, script: 'pipenv run yolk -V cognite-sdk | sort -n | tail -1 | cut -d\\  -f 2').trim()
            def currentVersion = sh(returnStdout: true, script: 'pipenv run python3 -c "import cognite; print(cognite.__version__)"').trim()

            println("This version: " + currentVersion)
            println("Latest pip version: " + pipVersion)
            if (env.BRANCH_NAME == 'master' && currentVersion != pipVersion) {
                stage('Release') {
                    sh("pipenv run twine upload --config-file /pypi/.pypirc dist/*")
                }
            }
        }
    }
}
