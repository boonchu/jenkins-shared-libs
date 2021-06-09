### Getting Started with Shared Library in Jenkins

* https://www.youtube.com/watch?v=Wj-weFEsTb0

### Using Resource Files from a Jenkins Shared Library

* https://www.youtube.com/watch?v=eV7roTXrEqg

### How to Run Shell in Jenkins

* https://www.youtube.com/watch?v=mbeQWBNaNKQ

### Examples

* https://github.com/pavankjadda/BookStore/blob/master/Jenkinsfile
* https://gist.github.com/merikan/228cdb1893fca91f0663bab7b095757c
* https://github.com/darinpope?tab=repositories

### Shared Libraries

* https://github.com/Perficient-DevOps/jenkins-shared-library

### Stackoverflow

* https://stackoverflow.com/questions/39832862/jenkins-cannot-define-variable-in-pipeline-stage
* https://stackoverflow.com/questions/53810475/how-to-dynamically-load-jenkins-pipeline-file-from-other-pipeline
* https://stackoverflow.com/questions/43494302/how-to-test-whether-a-jenkins-plugin-is-installed-in-pipeline-dsl-groovy
* https://stackoverflow.com/questions/39451345/using-credentials-from-jenkins-store-in-a-jenkinsfile
* https://support.cloudbees.com/hc/en-us/articles/226122247-How-to-customize-Checkout-for-Pipeline-Multibranch
* https://stackoverflow.com/questions/29742847/jenkins-trigger-build-if-new-tag-is-released

### labs

* Setup 'manage config' to provision settings.xml to integrate with Nexus repository

  - https://blog.sonatype.com/using-nexus-3-as-your-repository-part-1-maven-artifacts

- required Config File Provider Plugin
- Managed Jenkins -> Managed Files
- Add a new config nexus-maven-settings.xml -> add serverId nexus.dev
- Roll in the template 'settings-template.xml'
- Provision Jenkins credential for Nexus (nexus.dev, nexus username and password)
- Replace CONFIG_FILE_UUID in Jenkinsfile environment variable
