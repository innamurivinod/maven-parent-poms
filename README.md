# GantSign Maven Parent POMs

Parent Maven POMs with best practice configuration. The POMs are for general use and are not
company specific.

## Java

```xml
<project>
  ...
  <parent>
    <groupId>com.gantsign</groupId>
    <artifactId>java-parent</artifactId>
    <version>INSERT VERSION HERE</version>
    <relativePath />
  </parent>
  ...
</project>
```

## Kotlin

```xml
<project>
  ...
  <parent>
    <groupId>com.gantsign</groupId>
    <artifactId>kotlin-parent</artifactId>
    <version>INSERT VERSION HERE</version>
    <relativePath />
  </parent>
  ...
</project>
```

## Features

### Reproducible builds

The builds are made reproducible using the
[Reproducible Build Maven Plugin](https://github.com/Zlika/reproducible-build-maven-plugin). Making
your builds byte-for-byte reproducible helps various tools (e.g. Docker image layers) save on
storage if an artifact hasn't changed. It also provides a way of checking if an artifact has been
tampered with.

### Flattened POMs

The installed/deployed POMs are flattened using the
[Flatten Maven Plugin](https://www.mojohaus.org/flatten-maven-plugin). This installs/deploys a
version of the POM optimised for use as a dependency with any parent POMs inlined and configuration
only required during the build removed. This feature has to be enabled by adding the following to
your POM:
  
```xml
<project>
  ...
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>flatten-maven-plugin</artifactId>
      </plugin>  
      ...
    </plugins>
  </build>
  ...
</project>
```  

### Code auto-formatting

This project uses opinionated formatters for Java and Kotlin too avoid
[bikeshedding](https://en.wiktionary.org/wiki/bikeshedding). The Java parent POM uses the
[fmt-maven-plugin](https://github.com/coveooss/fmt-maven-plugin) that uses
[google-java-format](https://github.com/google/google-java-format) to format the code. The Kotlin
parent POM uses the [ktlint-maven-plugin](https://github.com/gantsign/ktlint-maven-plugin) that uses
[ktlint](https://github.com/pinterest/ktlint) to format the code. The POMs are formatted using the
[Sortpom Maven Plugin](https://github.com/Ekryd/sortpom). 

### Code linting

While auto-formatting can fix a lot, it can't fix every style issue for you. This project uses
linters to inform you of issues that need manual attention. The Java parent POM uses the
[maven-checkstyle-plugin](https://maven.apache.org/plugins/maven-checkstyle-plugin/) that uses
[Checkstyle](https://checkstyle.sourceforge.io) to lint the code. The Kotlin parent POM uses the
[ktlint-maven-plugin](https://github.com/gantsign/ktlint-maven-plugin) that uses
[ktlint](https://github.com/pinterest/ktlint) to lint the code.

### License headers

This project includes the
[License Maven Plugin](https://www.mojohaus.org/license-maven-plugin/usage.html) to add license
headers to your files. To use this plugin you need to configure it e.g.:

```xml
<project>
  ...
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>license-maven-plugin</artifactId>
        <configuration>
          <projectName>TODO</projectName>
          <!-- For a list of standard license names run: mvn -N license:license-list -->
          <!-- For proprietary licenses see 
               https://www.mojohaus.org/license-maven-plugin/examples/example-add-license.html -->
          <licenseName>TODO e.g. apache_v2</licenseName>
          <extraExtensions>
            <kt>java</kt>
            <kts>java</kts>
          </extraExtensions>
        </configuration>
        <executions>
          <execution>
            <id>config-files</id>
            <goals>
              <goal>update-file-header</goal>
            </goals>
            <phase>process-sources</phase>
            <configuration>
              <roots>${basedir}</roots>
              <includes>.editorconfig,.gitattributes,.travis.yml,pom.xml</includes>
              <extraExtensions>
                <editorconfig>properties</editorconfig>
                <gitattributes>properties</gitattributes>
                <yml>properties</yml>
              </extraExtensions>
            </configuration>
          </execution>
          <execution>
            <id>sources</id>
            <goals>
              <goal>update-file-header</goal>
            </goals>
            <phase>process-sources</phase>
          </execution>
        </executions>
      </plugin>
      ...
    </plugins>
  </build>
  
  <!-- The inception year is included in the licence header as the copyright year -->
  <inceptionYear>2018</inceptionYear>
  
  <!-- The organization name is included in the licence header as the copyright owner -->
  <organization>
    <name>TODO e.g. ACME LTD</name>
  </organization>

  <!-- This is the license metadata for the Maven project and should match the license of used
       with the license-maven-plugin. -->
  <licenses>
    <license>
      <name>TODO e.g. Apache License, Version 2.0</name>
      <url>TODO e.g. https://opensource.org/licenses/Apache-2.0</url>
      <distribution>repo</distribution>
      <comments>TODO e.g. A business-friendly OSS license</comments>
    </license>
  </licenses>
</project>
```

### Dependency analysis

It's quite easy to put a dependency in the wrong scope or to leave in a dependency after all usages
have been removed. This can slow down your builds and inflate the size of your deployables. It's
also common to rely on classes from your transitive dependencies that could disappear when the
dependency changes. So this project uses the
[Maven Dependency Plugin](https://maven.apache.org/plugins/maven-dependency-plugin) to check for
unused (possibly wrong scope) and undeclared dependencies.

### Maven Enforcer

Out of the box Maven builds can be a bit brittle.

* If you don't specify the plugin version Maven will default to the latest version. If the latest
  version is buggy or the API breaks compatibility then your build breaks. So it's best to use Maven
  Enforcer to require plugin versions be specified.
  
* Maven has a bonkers approach to picking between multiple dependency versions. It uses the nearest
  version in the dependency tree. This approach tends to leave you using buggy older dependency
  versions. It also possible that by adding a dependency you may inadvertently downgrade the version
  of another dependency. So it's best to use Maven Enforcer to require the dependency version to be
  specified if there's more than one version and that this be the highest version. This is achieved
  by specifying the dependency version in the `dependencyManagement` section. 

* Maven quite happily lets you build with dependencies containing duplicate classes (unless you're
  using Java 9+ modules). Maven has no notion of dependency substitutions e.g. `log4j-over-slf4j`
  and is poor at handling dependencies that change coordinates, so duplicate classes are common.
  It's entirely possible to use one class version in your IDE, one for the build and another at
  runtime. This compromises your tests and can make troubleshooting bugs really difficult. So it's
  best to use Maven Enforcer to ban duplicate classes. This typically requires you to exclude the
  conflicting transitive dependencies e.g. `log4j`. 

### Docker

This project includes the
[Jib Maven Plugin](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin) for
building Docker images of your applications. To use this plugin you need to configure it and
activate the appropriate Maven profile e.g.:

```xml
<project>
  ...
  <properties>
    <docker.image>TODO e.g. gcr.io/my-gcp-project/my-app</docker.image>
  </properties>
  ...
</project>
```

To build your application to your local Docker daemon activate the `docker-local` profile e.g:

```bash
mvn clean install -P docker-local
``` 

To build your application to your remote Docker registry activate the `docker-registry` profile e.g:

```bash
mvn clean install -P docker-registry
``` 

### Managed plugins

The following plugins have their version specified so builds are reproducible:

* com.coveo:fmt-maven-plugin
* com.github.ekryd.sortpom:sortpom-maven-plugin
* com.github.gantsign.maven:ktlint-maven-plugin
* com.google.cloud.tools:jib-maven-plugin
* io.github.zlika:reproducible-build-maven-plugin
* org.apache.maven.plugins:maven-checkstyle-plugin
* org.apache.maven.plugins:maven-clean-plugin
* org.apache.maven.plugins:maven-compiler-plugin
* org.apache.maven.plugins:maven-dependency-plugin
* org.apache.maven.plugins:maven-deploy-plugin
* org.apache.maven.plugins:maven-enforcer-plugin
* org.apache.maven.plugins:maven-failsafe-plugin
* org.apache.maven.plugins:maven-gpg-plugin
* org.apache.maven.plugins:maven-install-plugin
* org.apache.maven.plugins:maven-invoker-plugin
* org.apache.maven.plugins:maven-jar-plugin
* org.apache.maven.plugins:maven-javadoc-plugin
* org.apache.maven.plugins:maven-jdeps-plugin
* org.apache.maven.plugins:maven-jxr-plugin
* org.apache.maven.plugins:maven-plugin-plugin
* org.apache.maven.plugins:maven-project-info-reports-plugin
* org.apache.maven.plugins:maven-release-plugin
* org.apache.maven.plugins:maven-resources-plugin
* org.apache.maven.plugins:maven-shade-plugin
* org.apache.maven.plugins:maven-site-plugin
* org.apache.maven.plugins:maven-source-plugin
* org.apache.maven.plugins:maven-surefire-plugin
* org.apache.maven.plugins:maven-surefire-report-plugin
* org.codehaus.mojo:build-helper-maven-plugin
* org.codehaus.mojo:flatten-maven-plugin
* org.codehaus.mojo:jdepend-maven-plugin
* org.codehaus.mojo:license-maven-plugin
* org.jacoco:jacoco-maven-plugin
* org.jetbrains.kotlin:kotlin-maven-plugin
* org.sonatype.plugins:nexus-staging-maven-plugin

### Managed dependencies

The following dependencies have their version specified for convenience:

* com.google.code.findbugs:jsr305
* com.google.guava:guava
* io.github.microutils:kotlin-logging
* org.jetbrains:annotations
* org.jetbrains.kotlin:kotlin-compiler-embeddable
* org.jetbrains.kotlin:kotlin-reflect
* org.jetbrains.kotlin:kotlin-stdlib
* org.jetbrains.kotlin:kotlin-stdlib-common
* org.jetbrains.kotlin:kotlin-stdlib-jdk8
* org.slf4j:jcl-over-slf4j
* org.slf4j:jul-to-slf4j
* org.slf4j:log4j-over-slf4j
* org.slf4j:slf4j-api
* org.slf4j:slf4j-simple
* org.projectlombok:lombok
* ch.qos.logback:logback-classic
* io.mockk:mockk
* io.mockk:mockk-dsl-jvm
* junit:junit
* org.assertj:assertj-core
* org.assertj:assertj-guava
* org.jetbrains.kotlin:kotlin-test
* org.jetbrains.kotlin:kotlin-test-common
* org.jetbrains.kotlin:kotlin-test-junit
* org.junit.jupiter:junit-jupiter-api
* org.junit.jupiter:junit-jupiter-engine
* org.mockito:mockito-core
* org.mockito:mockito-junit-jupiter


## License

This software is licensed under the terms in the file named "LICENSE" in the
root directory of this project. This project has dependencies that are under
different licenses.

## Author Information

John Freeman

GantSign Ltd.
Company No. 06109112 (registered in England)