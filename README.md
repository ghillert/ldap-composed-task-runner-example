# LDAP Security and Composed Task Runner Example

This example provides a quick glance on how to run Compose Tasks on
Spring Cloud DAta Flow with enabled LDAP security.

This repository provide an embedded LDAP server, running on *port 40000*
with pre-configured users. In this example we will use the following user:

- username: joe
- password: joespassword

## Build + Start LDAP Server

```bash
$ git clone https://github.com/ghillert/ldap-composed-task-runner-example.git
$ ./mwnw clean package
$ java -jar target/ldapserver-0.0.1-SNAPSHOT.jar
```

## Download + Start Spring Cloud Data Flow

```bash
wget https://repo.spring.io/milestone/org/springframework/cloud/spring-cloud-dataflow-server-local/1.7.0.RC1/spring-cloud-dataflow-server-local-1.7.0.RC1.jar
wget https://repo.spring.io/milestone/org/springframework/cloud/spring-cloud-dataflow-shell/1.7.0.RC1/spring-cloud-dataflow-shell-1.7.0.RC1.jar
```

Create a file `application.yml` with the following contents:

```yaml
security:
  basic:
    enabled: true
spring:
  cloud:
    dataflow:
      security:
        authentication:
          ldap:
            enabled: true
            url: ldap://localhost:40000
            managerDn: uid=bob,ou=people,dc=springframework,dc=org
            managerPassword: bobspassword
            userSearchBase: ou=otherpeople,dc=springframework,dc=org
            userSearchFilter: uid={0}
            groupSearchFilter: member={0}
            groupRoleAttribute: cn
            groupSearchBase: ou=groups,dc=springframework,dc=org
```

## Configure and run a Composed Task

First start the Spring Cloud Data Flow Shell:

```bash
java -jar spring-cloud-dataflow-shell-1.7.0.RC1.jar --dataflow.username=joe --dataflow.password=joespassword
```

Now we need to import the Composed Task Runner and the Spring Cloud Task App Starters:

```bash
dataflow:> app import http://bit.ly/Dearborn-GA-task-applications-maven
```

If you want to import _just_ the Composed Task Runner applications:

```bash
dataflow:> app register --name composed-task-runner --type task --uri  maven://org.springframework.cloud.task.app:composedtaskrunner-task:2.0.0.RELEASE
```

It is important that use the latest task app starters, so we end up having at
least _Composed Task Runner_ version `2.0.0.RELEASE`. The earlier versions
had [short-comings](https://github.com/spring-cloud-task-app-starters/composed-task-runner/issues/41)
in regards to security. Therefore, don't use the app starters from the *Clark*
release train.

Create + Run the Composed Task:

```bash
dataflow:> task create my-composed-task --definition "timestamp && timestamp-batch"
dataflow:> task launch my-composed-task --arguments "--dataflow-server-username=joe --dataflow-server-password=joespassword"
```
