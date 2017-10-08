---
layout: post
title: Creating an aggregated SonarAube report for multiple Maven projects
enabledisqus: true
---

SonarQube integrates pretty well with Maven artifacts and we use our Jenkins build jobs to send Sonar reports to the SonarQube server. SonarQube neatly creates a project for each Maven artifact out of the box. In my current project, we use SonarQube dashboards on a large screen which we go through each morning during standup, where we identify any new issues that SonarQube reports. This works really well, but since SonarQube only shows one dashboard per artifact, we only have time to go through one or two dashboards per standup. My team has responsibility of over 15 different small application, and we miss having a single dashboard that collects all our applications.

I asked Google for help, but I only found payed SonarQube plugins which I was unsure would solve our problem. Luckily, I was able to create solution for our need using Jenkins Pipeline and the fact that Sonar can group projects by a _branch_ name. The idea is simple: I create a temporary pom file with the projects I want to aggregate on as sub-modules. Then I run the Maven Sonar plugin on the temporary pom which create the aggregated SonarQube project.

The following Jenkins Pipeline script does the job:

{% highlight groovy %}
def root_pom = """
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>no.qadeer</groupId>
  <artifactId>my-aggregated-report</artifactId>
  <name>My Aggregated Report</name>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>pom</packaging>

 <modules>
   <module>project1</module>
   <module>project2</module>
   <module>project3</module>
 </modules>
</project>"""



node {
    stage('create-pom') {
        sh "echo '" + root_pom + "' > pom.xml"
    }

    stage('checkout projects') {
        dir(project1) {
            git branch: 'develop', url: 'ssh://git@git.qadeer.no:7999/project1.git'
        }

        dir(project2) {
            git branch: 'develop', url: 'ssh://git@git.qadeer.no:7999/project2.git'
        }

        dir(project3) {
            git branch: 'develop', url: 'ssh://git@git.qadeer.no:7999/project3.git'
        }

    }

    stage('run sonar') {
        withSonarQubeEnv {
            sh "mvn -B clean -Dsonar.branch=my-aggregates org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar"
        }
    }

}
{% endhighlight %}


I first create a temporary parent `pom.xml` on the fly which lists the projects I want an aggregate report on as sub-modules. Then, for each project, I checkout the projects in subfolders. Finally, I run the sonar plugin with the `sonar.branch` flag.

The flag is important because this will separate your original project dashboard from the copy aggregate project. You might wonder why it is not easier to simply write the report to the original project? If you try without the flag, SonarQube will refuse to process the reports sent by Maven and complain about duplicate key and that the project already exists:


    The project "com.foo:bar-submodule-1" is already defined in SonarQube but not as a module of project "com.foo:bar". If you really want to stop directly analysing project "com.foo:bar-submodule-1", please first delete it from SonarQube and then relaunch the analysis of project "com.foo:bar"


 By adding the `sonar.branch` flag, SonarQube prefixes the key with the flag, creating a new project. You could delete the original project and then run the pipeline to avoid the error, but then you will loose all your history. This approach does create a duplicate project, but that is livable. The best would of course be if SonarQube updated the original project, but I guess the way SonarQube groups projects using their parents require that an existing project cannot have two different parents.




