pipeline {
  agent any
  stages {
    stage('Build source') {
      steps {
        git 'https://github.com/ubports/oxide-tools.git'
        sh '''
        VER="1.22"

        ./oxide-tools/fetch_depot_tools
        export PATH=$PATH:$(realpath ~/.cache/depot_tools)
        ./oxide-tools/fetch_oxide -b $VER ./

        mv src source
        mv debian source

        export SKIP_MOVE=true
        build-source.sh
        '''
        stash(name: 'source', includes: '*.gz,*.bz2,*.xz,*.deb,*.dsc,*.changes,*.buildinfo,lintian.txt')
        cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenSuccess: true, cleanWhenUnstable: true, deleteDirs: true)
      }
    }
    stage('Build binary - armhf') {
      steps {
        node(label: 'xenial-arm64') {
          unstash 'source'
          sh '''export architecture="armhf"
export BUILD_ONLY=true
/usr/bin/build-and-provide-package'''
          stash(includes: '*.gz,*.bz2,*.xz,*.deb,*.dsc,*.changes,*.buildinfo,lintian.txt', name: 'build')
          cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenSuccess: true, cleanWhenUnstable: true, deleteDirs: true)
        }

      }
    }
    stage('Results') {
      steps {
        unstash 'build'
        archiveArtifacts(artifacts: '*.gz,*.bz2,*.xz,*.deb,*.dsc,*.changes,*.buildinfo', fingerprint: true, onlyIfSuccessful: true)
        sh '''export architecture="armhf"
/usr/bin/build-repo.sh'''
      }
    }
    stage('Cleanup') {
      steps {
        cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenSuccess: true, cleanWhenUnstable: true, deleteDirs: true)
      }
    }
  }
}
