# Logging

## What is Logging
- act of publishing diagnostics information at certain points of program execution
- you can write files to log files or console to help you understand what the application is doing
- a crude way to do logging would be:
```java
private void truncateTable(String tableName) {
    System.out.format("[WARN] truncating table `%s`%n", tableName); 
    db.truncate(tableName);
}
```
- there are several drawbacks to this approach

## Why log4j2
- it is a versatile, industrial-grade java logging framework
- helps with
  - enhancing the message with additional info eg timestamp, file, class and method name, line number, host, severity ect
  - formatting the message according to given layout
  - writing the message to various targets using an appender e.g console, file, socket, database, queue etc
  - filtering messages to be written eg filter by severity, content

## Components
- logging api - log4j API
- reference implementation - log4j core
- logging bridges - enables log4j core to consume from foreign logging APIs

### Logging API
- interface your code or your dependencies directly logs against
- required at compile time
- implementation agnostic to ensure that your application can write logs but is not tied to specific logging implementation
- other examples
  - log4j API
  - slf4j
  - JUL - Java Logging
  - JCL - Apache Commons Logging
  - JPL - Java Platform Logging
  - JBoss Logging

### Logging Implementation
- required at runtime
- can be changed without recompiling your software
- examples:
  - Log4j Core
  - JUL - Java Logging
  - Logback

### Logging Bridge
- logging implementations accept input from a single logging API of their preference.
- log4j core from log4j API, logback from SLF4J etc
- logging bridge is a simple logging implementation of a logging API that forwards all messages to a foreign logging API
- example
  - log4j-slf4j2-impl bridges slf4j calls to log4j API and effectively enables log4j core to accept input from slf4j

## Installation
- we will need a BOM(Bill of Materials) to manage the versions of the dependencies
- this way, we will not need to provide the version for each log4j module explicitly
```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-bom</artifactId>
      <version>2.24.1</version>
      <scope>import</scope>
      <type>pom</type>
    </dependency>
  </dependencies>
</dependencyManagement>
```

## How to log use Log4j API
- to log, you need a **Logger** instance which you will retrieve from the **LogManager**
- these are all part of the **log4j-api** module which you can install as follows
```xml
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-api</artifactId>
  <version>${log4j-api.version}</version>
</dependency>
```
- you can use the **Logger** instance to log using methods like `info()`, `warn()`, `error()` etc
- these methods are named after the log levels they represent
- log message can contain placeholders written as `{}` that will be replaced by the arguments passed to the method
```java
import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.LogManager;

public class DbTableService {

    private static final Logger LOGGER = LogManager.getLogger(); 

    public void truncateTable(String tableName) throws IOException {
        LOGGER.warn("truncating table `{}`", tableName); 
        db.truncate(tableName);
    }

}
```

## Best Practices
- don't use toString()
- pass exception as the last argument
  - don't call Throwable#printStackTrace() - it can leak sensitive information
  - don't use Throwable#getMessage() - prevents the log event from getting enriched with the exception
  - don't provide both Throwable#getMessage() and Throwable itself
- don't use string concatenation
  - use Suppliers to pass computationally expensive arguments
    ```java
    /* BAD! */ LOGGER.info("failed for user ID `{}` and role `{}`", userId, db.findUserRoleById(userId));
    ```
    - if the info log will be discarded anyway, this could be a significant bottleneck
    - oldschool way of solving this is to level-guard the log statement
    ```java
    /* OKAY */ if (LOGGER.isInfoEnabled()) { LOGGER.info(...); } 
    ```
    - while this would work for cases where the message can be dropped due to insufficient level, this approach is still prone to other filtering cases
    - use supplier to pass arguments containing computationally expensive items
      ```java
      /* GOOD */ LOGGER.info("failed for user ID `{}` and role `{}`", () -> userId, () -> db.findUserRoleById(userId));
      ```
      ```java
      /* GOOD */ LOGGER.info(() -> new ParameterizedMessage("failed for user ID `{}` and role `{}`", userId, db.findUserRoleById(userId)));
        ```