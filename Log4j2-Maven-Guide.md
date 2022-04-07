# Quick guide to use log4j2 in a Maven project
---
## 1) Maven Dependencies
### Exclude other logging frameworks
In the **pom.xml** file of your project, delete/comment every other possible dependency to other logging frameworks. This is because if a logging hierarchy is present on the server you're deploying you project on, there is a chance that another logging framework has a higher level in the hierarchy and will take over log4j2. 
### Add log4j2 dependencies
Now add the log4j2 dependiencies in the **pom.xml** file of your project. If your project consists of multiple modules, adding these dependencies to the most external project's pom.xml file should be enough.
In the dependencies section of the pom add:

		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-api</artifactId>
			<version>2.17.2</version>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-core</artifactId>
			<version>2.17.2</version>
		</dependency>
You can always check the dependencies or the latest version to use here: https://logging.apache.org/log4j/2.x/maven-artifacts.html#Using_Log4j_in_your_Apache_Maven_build

## 2) Add the configuration file
Log4j2 needs a configuration file where you can specify how the logging framework has to behave when it's called.  
This file can be a **.properties** file or a **.xml** file. In this guide, we will see how to use the .xml file.
This file must be named **log4j2.xml** and can be located anywhere in the application's classpath, even though is usually found in the **src/main/resources** . In a multi module project it's the same, but it needs to be inside the module building the entire project.
### Configuration file example
Here is a simple example of **log4j2.xml** file. This simple configuration defines a simple appender used by the root logger to log on the console.
                   
    <Configuration status="warn">  
		<Appenders>  
			<Console name="console" target="SYSTEM_OUT">  
				<PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n" />  
			</Console>  
		</Appenders>  
		<Loggers>  
			<Root level="info" additivity="false">  			
				<AppenderRef ref="console" />
			</Root>  
		</Loggers>  
    </Configuration>

You can find examples for more complex configurations (for example for logging on files), at this link:
https://howtodoinjava.com/log4j2/log4j2-xml-configuration-example/
or in the official Apache Log4j2 manual:
https://logging.apache.org/log4j/2.x/manual/configuration.html
## 3) Using log4j2 from your code:
After everything is properly configured, log4j2 is ready to be used inside the application code.
Here is a simple example of using a log4j2 logger.  The **LogManager**'s getLogger() method returns a **Logger** object that can be used for logging whenever it is needed inside the class;
		 

	import org.apache.logging.log4j.LogManager;
	import org.apache.logging.log4j.Logger;

	public class Log4j2Test {
    private static final Logger logger = LogManager.getLogger();

	    public Log4j2Test(){
	        logger.info( "Hello World!" );
	    }
	}
How and where will the application log? It depends on how the **log4j2.xml** file is configured.
You can find more examples for configuring multiple loggers, appenders, log levels and more at the following link:
https://logging.apache.org/log4j/2.x/manual/

## References
* [Log4j2 official website](https://logging.apache.org/log4j/2.x/)
* [Configuration Example - howtodoinjava](https://howtodoinjava.com/log4j2/log4j2-xml-configuration-example/)