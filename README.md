[![Build Status](https://travis-ci.org/rewe-digital-incubator/frost.svg?branch=master)](https://travis-ci.org/rewe-digital-incubator/frost) 

# FROST

_Making GUI system tests extremely cool!_
  
FROST enables you to run self-contained, multi-browser Galen-based Selenium tests within your build. External dependencies(e.g. Databases, Asset Servers etc.) are provided as Docker containers and can be defined just via a docker-compose file.

FROST is implemented as a Gradle Plugin. But this does not require your project being a Gradle project. It simple does not matter at all how your project is being built.

![Logo](docs/logo.png)


## Getting Started

### Applying the Plugin

The plugin is available via [Gradle Plugin Portal](https://plugins.gradle.org/). Just apply it in your `build.gradle` as follows:
```
buildscript {
    dependencies {
        classpath('org.rewedigital:frost:0.5.2')
    }
}
apply plugin: "org.rewedigital.frost"
```


### Configuration

You can use the following configuration in your `build.gradle` file:
```
frost {
    // FROST working directory. Relative directories are being considerd relative to the root project
    // directory. Default is 'frost'.
    frostWorkingDirectory = "uiTest"
    
    // Directory in which to store the cached Galen binary. Relative directories are being considerd
    // relative to the project directory. Default is '<USER_HOME>/.frost'.
    frostCacheDirectory = "uiTest"
    
    // The Galen version to use, default is "2.4.4".
    galenVersion = '2.4.4'
    
    // The URL where to download the Galen binary, default is
    // "https://github.com/galenframework/galen/releases/download/galen-${galenVersion}/galen-bin-${galenVersion}.zip".
    galenDownloadUrl = "https://my-company-bin-repository/galen/galen-special-version.zip"

    // Which browsers to use, default is ['chrome', 'firefox'].
    // Supported browsers are 'chrome' and 'firefox'.
    browsers = ["chrome"]
    
    // Which Docker images to use for the browsers.
    // Default is selenium/standalone-chrome:latest and selenium/standalone-firefox:latest.
    browserImages = [ chrome: 'selenium/standalone-chrome:3.13.0']
    
    // Directory containing the Galen test suites. Relative directories are being considerd relative to
    // the project directory. Default is 'src/uiTest/frost/tests'.
    testSuitesDirectory = "src/uiTest/frost/tests"
    
    // Whether to search for all ".test" files recursively in the "testSuitesDirectory", default is false.
    recursive = true

    // Comma separated list of test groups to be executed, default is all test groups. If left empty
    // all test groups are executed.
    testGroups = "ci"
   
    // Amount of threads per browser for running tests in parallel, default is 1.
    numberOfParallelTests = 2

    // Tag of the SUT image to be tested, default is 'latest'.
    sutTag = "${applicationVersion}"
    
    // Port (internal) used by the SUT, default is 8080.
    sutPort = 8081
    
    // Path of the endpoint to be queried in order to detect if the SUT is up and running.
    // Default is '/admin/healthcheck'.
    // The endpoint must respond with status code 200 if and only if the SUT is ready.
    sutHealthCheckEndpointPath = "/health"
   
    // The maximum time to wait (in minutes) for the SUT healthcheck to signal UP after service start.
    // Default is 5.
    sutReadinessTimeoutInMinutes = 10

    // Docker compose file describing the environment of the SUT including all of its dependencies.
    // Relative directories are being considerd relative to the project directory.
    // Default is 'docker-compose.yml'.
    // You should omit ports, s.t. the plugin will chose a random free port.
    composeFile = 'docker-compose.frost.yml'

    // Docker compose file to describe the environment of the browsers.
    // Relative directories are being considerd relative to the build directory.
    // Default is 'docker-compose.override.frost.yml'.
    // There is no need to manage this manually, it is just for internal use.
    composeOverrideFile = 'docker-compose.override.frost.yml'
    
    // When set, it is being used as the value for the COMPOSE_PROJECT_NAME environment variable when
    // starting the docker-compose application. This affects the prefix of the docker container names.
    // E.g. using the projectName "foo" will result in container names "foo_sut_1", "foo_proxy_1" etc. 
    // You should use this when you are encountering issues with concurrent runs of other projects' FROST
    // tests on the same machine. 
    projectName = 'frost_myproject'
    
    // Whether the requests to the SUT should be routed through a proxy (wiremock), default is false.
    // This can be useful to add or modify HTTP request headers that your SUT may rely on, as Galen does
    // not seem to support this directly. 
    useProxy = true
    
    // When using the proxy, this is the directory where the wiremock configuration files are based.
    // Relative directories are being considerd relative to the project directory. Default is 'frost'.
    proxyConfigurationDirectory = 'uiTest/wiremock-config'

    // Whether or not failing Frost tests or framework errors should let the Gradle task/build fail.
    // Default is true.
    failBuildOnErrors = false
}
```


### Defining the System Under Test

To define the external dependencies of your System Under Test (SUT) use a docker-compose file like this:
```
version: '2'

services:
  db:
    image: docker-registry.mycompany.com/myservice-test-db:latest
  assets:
    image: sebp/lighttpd
    volumes:
      - ./npm_dist:/var/www/localhost/htdocs
  sut:
    depends_on:
      - "db"
      - "assets"
    image: docker-registry.mycompany.com/myservice:${TAG}
    command: ["-c", "/opt/wait-for.sh db:5432 && /usr/sbin/java-service start"]
    entrypoint: ["/bin/sh"]
    environment:
        - SYSTEM_MEMORY=1024
        - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/
        - SPRING_DATASOURCE_USERNAME=postgres
        - SPRING_DATASOURCE_PASSWORD=
        - ENVIRONMENT_ASSETBASEURL=http://assets:80/
```


### Example Run
To run simply execute:
```
./gradlew frostRun
```


## Local build of the Plugin

For local development use the task

```
./gradlew publishToMavenLocal
``` 

This will build the plugin and publish it to your local Maven Repository (~/.m2). You can reference the plugin in your service via
```
buildscript {
    repositories {
        mavenLocal()
    }
    dependencies {
        classpath('org.rewedigital:frost:0.5.2')
    }
}
apply plugin: "org.rewedigital.frost"
```

## License
The MIT license (MIT)

Copyright (c) 2017-2018 REWE Digital GmbH

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
