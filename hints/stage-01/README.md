# Hints for Tutorial stage 01

## Configure the config server with a local git config-repository

Try to configure the config server to use a local config-repository first. You can use the official [Spring Cloud Config documentation](https://cloud.spring.io/spring-cloud-config/spring-cloud-config.html) and/or the [Baeldung Spring Cloud Config Tutorial](http://www.baeldung.com/spring-cloud-configuration) for reference.

Steps:

**Config Server**

1. Add yaml dependencies to the config-server ```build.gradle```
    ```
    compile('org.yaml:snakeyaml')
    ```
2. Enable the config server by adding annotation ```@EnableConfigServer``` on the class marked with ```@SpringBootApplication``` (```ConfigApplication```)
3. Create file ```application.yml``` in the resources folder (you can remove existing ```.properties``` files)
4. Configure the config-server in the config server's application.yml you just created:

    ```YAML
    spring:
      cloud:
        config:
          server:
            git:
              uri: file:///${user.home}/Desktop/customer
    server:
      port: 8888
    ```

**Config Repository**

5. Create a folder that matches the pattern of the uri you provided above (```file:///${user.home}/Desktop/customer``` >> just create a folder ```customer``` on your Desktop)
6. run ```git init``` in the folder you created
7. Add the file ```customer-dev.yml``` to the folder. This will be the configuration file that the client application (the customer application) pulls upon startup.
8. Add the configuration for the new port to the ```customer-dev.yml``` file (instead of running the customer application on default port 8080 we run it on port 8081):

    ```YAML
    server:
      port: 8081
    ```

9. Commit the file ```customer-dev.yml``` using ```git commit```
10. Test the configuration by starting the config-server and navigating to ```http://localhost:8888/customer/dev```

**Client Configuration**

11. Add yaml and bootstrap dependencies to the client project (the customer project)

    ```
    dependencies {
    	compile('org.springframework.boot:spring-boot-starter-web')
    	compile('org.springframework.cloud:spring-cloud-starter-config')
        compile('org.springframework.boot:spring-boot-starter-actuator')
    	compile('org.yaml:snakeyaml')
        testCompile('org.springframework.boot:spring-boot-starter-test')
    }
    dependencyManagement {
	    imports {
		    mavenBom "org.springframework.cloud:spring-cloud-dependencies:Camden.SR5"
	    }
    }
    ```

12. Create file ```bootstrap.yml``` in the resources folder (you can remove existing ```.properties``` files)
13. Configure the client to find the config-server by in the ```bootstrap.yml```:

    ```YAML
    spring:
      application:
        name: customer
      profiles:
        active: dev
      cloud:
        config:
          uri: http://localhost:8888
    ```

14. Test the setup by starting the customer-service. If everything works correct it will start up on port ```8081``` now.

## Configure the config-server to use a remote config-repository

Once you are done with the local setup you can reconfigure the config server to use a remote config-repository instead of your local repository. To do so you will have to adapt the config server's ```application.yml```:

```YAML
spring:
  cloud:
    config:
      server:
        git:
          uri: [PATH_TO_REMOTE_REPOSITORY]
          searchPaths: [FOLDER_IN_REMOTE_REPOSITORY]
server:
  port: 8888
```

Example configuration with the BankingInTheCloud-Tutorials repository:

**application.yml** (config project)

```YAML
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/senacor/BankingInTheCloud-Tutorials
          searchPaths: config-repo
server:
  port: 8888
```

Since the ```config-repo``` folder is on a specific branch within the BankingInTheCloud-Tutorials repository you have to specify the branch as ```label``` in the ```bootstrap.yml``` of the customer application to make the setup work:

**bootstrap.yml** (customer project)

```YAML
spring:
  application:
    name: customer
  profiles:
    active: dev
  cloud:
    config:
      uri: http://localhost:8888
      label: Stage-01-SpringCloudConfig
```
