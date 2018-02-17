# Software Engineering Course

## Prerequisites
* Install [Talan](https://github.com/project-talan/talan-core)
* Install toolset
  ```
  > tln install java,maven,nodejs,docker,docker-compose
  ```
* Clone repository with helpers
  ```
  > cd ~
  > mkdir projects
  > cd projects
  > git clone https://github.com/swe-course/swec-content.git
  ```

## Nexus
* Up Nexus instance
  ```
  > cd ~/projects
  > cd swec-content/nexus
  > ./nexus-up.sh -d
  ```
* Access point **http://\<host-ip-address\>:8081**, user/pass **admin/admin123**
* Create new maven2 hosted repository, name: **saas-template**
  ![](https://raw.githubusercontent.com/swe-course/swec-content/master/imgs/nexus-maven-repo.png)

## Github
* Create personal access token with **repo**, **admin:repo_hook**
  ![](https://raw.githubusercontent.com/swe-course/swec-content/master/imgs/github-token.png)


## SonarQube
* Up SonarQube instance
  ```
  > cd ~/projects
  > cd swec-content/sonarqube
  > ./sonarqube-up.sh -d
  ```
* Access point **http://\<host-ip-address\>:9000**, user/pass **admin/admin**
* Install "Github" plugin
  ![](https://raw.githubusercontent.com/swe-course/swec-content/master/imgs/sonar-github.png)
* Generate access token
  ![](https://raw.githubusercontent.com/swe-course/swec-content/master/imgs/sonar-token.png)


## Jenkins
* Install
  ```
  > tln install jenkins
  ```
* Access point **http://\<host-ip-address\>:8080**
* Apply fix(es) for **"ALPN callback dropped: SPDY and HTTP/2 are disabled. Is alpn-boot on the boot class path?"**
  * Check your Java version 
    ```bash
    > java -version
    ```
    
    > openjdk version "1.8.0_151"
    
    > OpenJDK Runtime Environment (build 1.8.0_151-8u151-b12-0ubuntu0.16.04.2-b12)
    
    > OpenJDK 64-Bit Server VM (build 25.151-b12, mixed mode)
    
  * Find corresponding alpn boot library
  
    ```
    https://www.eclipse.org/jetty/documentation/9.4.x/alpn-chapter.html#alpn-versions
    ```
    | OpenJDK version | ALPN version |
    | --- | --- |
    | 1.8.0u151 | 8.1.11.v20170118 |
    
  * Copy link to the corresponding ALPN jar from 'ALPN version' folder
    ```
    http://central.maven.org/maven2/org/mortbay/jetty/alpn/alpn-boot/
    ```
    > http://central.maven.org/maven2/org/mortbay/jetty/alpn/alpn-boot/8.1.11.v20170118/alpn-boot-8.1.11.v20170118.jar
    
  * Goto JVM folder
    ```
    > cd /usr/lib/jvm
    ```
  * Download jar file
    ```
    > wget http://central.maven.org/maven2/org/mortbay/jetty/alpn/alpn-boot/8.1.11.v20170118/alpn-boot-8.1.11.v20170118.jar
    ```
  * Goto folder
    ```
    > cd /etc/default
    ```
  * Open for editing **jenkins** file
  * Modify property **JAVA_ARGS** variable
    from
    ```
    JAVA_ARGS="-Djava.awt.headless=true"
    ```
    too
    ```
    JAVA_ARGS="-Djava.awt.headless=true -Xbootclasspath/p:/usr/lib/jvm/alpn-boot-8.1.11.v20170118.jar"
    ```
  * Restart Jenkins
    ```
    systemctl stop jenkins && systemctl start jenkins
    ```

### Install plugins
* "GitHub Pull Request Builder"
* "SonarQube Scanner"
* "Sonar Quality Gates"
* "Pipeline Utility Steps"
* "HTTP Request"

### Configure plugins
* Configure "SonarQube servers" instance, name **SonarQube**
  ![](https://raw.githubusercontent.com/swe-course/swec-content/master/imgs/jenkins-sonar.png)
* "GitHub" instance + Github access credentials using created token
  ![](https://raw.githubusercontent.com/swe-course/swec-content/master/imgs/jenkins-github.png)
* ? "Quality Gates - Sonarqube"
* "GitHub Pull Request Builder"
  ![](https://raw.githubusercontent.com/swe-course/swec-content/master/imgs/jenkins-ghprb.png)

### Configure tools
* Configure "SonarQube Scanner", name - **SonarQube Scanner 3.0.3.778**
  ![](https://raw.githubusercontent.com/swe-course/swec-content/master/imgs/jenkins-tools-sonar-scanner.png)

## Clone template **https://github.com/swe-course/saas-template**

## Create new pipeline job
* Specify your fork url
  ![](https://raw.githubusercontent.com/swe-course/swec-content/master/imgs/jenkins-pipeline-repo.png)
* Add job parameters

| Parameter name | Value |
| --- | --- |
| SONARQUBE_SERVER | **SonarQube** |
| SONARQUBE_SCANNER | **SonarQube Scanner 3.0.3.778** |
| SONARQUBE_ACCESS_TOKEN | - |
| GITHUB_ACCESS_TOKEN | - |
| NEXUS_HOST | **http://\<host-ip-address\>:8081** |
| NEXUS_REPO | **saas-template** |
| NEXUS_USER | **admin** |
| NEXUS_PASS | **admin123** |
| SERVICES_GJ_PORT | **8182** |
* Configure "GitHub Pull Request Builder"
  ![](https://raw.githubusercontent.com/swe-course/swec-content/master/imgs/jenkins-pipeline-ghprb.png)
  * Add your user into WhiteList
* "GitHub hook trigger for GITScm polling"  
* Set pipeline definitions
  * Add additional branch
    ```
    ${sha1}
    ```
  * Use git repo Refspec:
    ```
    +refs/heads/*:refs/remotes/origin/* +refs/pull/*:refs/remotes/origin/pr/*
     ```
  ![](https://raw.githubusercontent.com/swe-course/swec-content/master/imgs/jenkins-pipeline-definition.png)

## Gonfigure branch(es)
* Mark master branch as protected
