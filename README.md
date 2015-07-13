Using S3 as a maven repository with Gradle
==========================================

The Amazon S3 service can be easily used as a Maven (or Ivy) repository with Gradle.

Advantages
----------
* Cheap
* Secure
* Online

Disadvantages
-------------
* Lacks any administration, search
* Snapshots must be pruned manually

Setup
-----
Create an S3 bucket with any name. You can also re-use existing bucket and simply create a folder for your dependencies.

Add the following to your Gradle file to enable download of dependencies from S3:

```gradle
apply plugin: 'maven'

repositories {
    maven {
        name "name your repo for later use"
        url "s3://bucket-name/maven"
        credentials(AwsCredentials) {
            accessKey "aws access key"
            secretKey "aws secret key"
        }
    }
}
```

To be able to publish artifacts to S3, you can use uploadArchives or the new maven-publish plugin. The following shows
how to switch between two repositories and how to reference previously defined repositories. The switching works very well
with [Gradle Release Plugin](https://github.com/researchgate/gradle-release).

```gradle
apply plugin: 'maven-publish'

publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
    repositories {
        add project.version.endsWith('-SNAPSHOT') ? project.repositories.s3RepoSnapshots : project.repositories.s3RepoInternal
    }
}
```

To include source jars, test jars or other archives, e.g. from distribution plugin, add this to the publication. 
Group Id, Artifact Id can be set here as well. 

```gradle
apply plugin: 'maven-publish'

publishing {
    publications {
        maven(MavenPublication) {
            groupId `group`
            artifactId 'artifact'
            from components.java
            artifact sourceJar {
                classifier "sources"
            }
            artifact testJar {
                classifier "test"
            }
            artifact distZip {
                classifier "zip"
            }
        }
    }
}
```

Example
-------
This repository contains an example of the S3 use. Rename `gradle.properties.template` to `gradle.properties` and add
your own S3 config.

Migration from other maven repositories
---------------------------------------
I have successfully migrated Apache Archiva to S3 simply by copying the whole directory structure from Archiva's data 
folder to S3. Any maven repository should be possible to migrate the same way.

Future work
-----------
Pruning of the snapshots on release or scheduler. 