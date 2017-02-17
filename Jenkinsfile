#!/usr/bin/env groovy

/**
 * Build Creator feed packages
 *
 * Configure as needed, pull in the feeds and run the build.
 * Overrides Creator feed with local clone.
 *
 */

def creatorPackages = [
    'awalwm2m',
    'bit-bang-gpio',
    'gcc53',
    'info-daemon',
    'letmecreate',
    'onboarding-scripts',
    'pickle',
    'provisioning-daemon',
    'pyletmecreate',
]

properties([
    buildDiscarder(logRotator(numToKeepStr: '30')),
    parameters([
        stringParam(defaultValue: 'http://downloads.creatordev.io/openwrt/latest/pistachio/marduk/OpenWrt-SDK-v1.1.0-pistachio_gcc-5.3.0_musl-1.1.15.Linux-x86_64.tar.bz2',
            description: 'OpenWrt SDK tarball to use', name: "SDK_TARBALL"),
    ])
])

node('docker && imgtec') {
    def docker_image
    stage('Prepare Docker container') {
        docker_image = docker.image "imgtec/creator-builder:latest"
    }
    docker_image.inside {
        stage('Prepare') {
            // checkout Creator-feed to a sub directory
            checkout([$class: 'GitSCM',
                userRemoteConfigs: scm.userRemoteConfigs,
                branches: scm.branches,
                doGenerateSubmoduleConfigurations: scm.doGenerateSubmoduleConfigurations,
                submoduleCfg: scm.submoduleCfg,
                browser: scm.browser,
                gitTool: scm.gitTool,
                extensions: scm.extensions + [
                    [$class: 'CleanCheckout'],
                    [$class: 'PruneStaleBranch'],
                    [$class: "RelativeTargetDirectory", relativeTargetDir: "${WORKSPACE}/Creator-feed"],
                ],
            ])

            // Download and extract Creator SDK
            sh "wget -qO- ${params.SDK_TARBALL} | tar -xvj --strip-components 1"

            // Enable signed package indexing
            sh "echo '\nconfig SIGNED_PACKAGES\n\tbool\n\tdefault y\n' >> Config.in"

            // Override Creator feed with local clone
            sh "echo 'src-link creator ../Creator-feed' >> feeds.conf.default"

            // Update and install all feeds
            sh "cat Config.in feeds.conf.default"
            sh "scripts/feeds update -a && scripts/feeds install -a"
        }
        stage('Build') {
            // Attempt to build quickly and reliably
            // TODO Issue#32: build specific package only based on PR or commit
            for (item in creatorPackages) {
                try {
                    sh "make package/${item}/compile -j4 V=s"
                } catch (hudson.AbortException err) {
                    // TODO BUG JENKINS-28822
                    if(err.getMessage().contains('script returned exit code 143')) {
                        throw err
                    }
                    echo 'Parallel build failed, attempting to continue in single threaded mode'
                    sh "make package/${item}/compile -j1 V=s"
                }
            }
            // Add package signing key
            withCredentials([
                [$class: 'FileBinding', credentialsId: 'opkg-build-private-key', variable: 'PRIVATE_KEY'],
            ]){
                sh "cp ${env.PRIVATE_KEY} key-build"
                sh "make package/index -j1 V=s"
                sh "rm key-build"
            }
        }
        stage('Upload') {
            // only archive Creator feed packages and signed index
            archiveArtifacts 'bin/pistachio/packages/creator/*'
            // clean up the workspace to save space
            deleteDir()
        }
    }
}

