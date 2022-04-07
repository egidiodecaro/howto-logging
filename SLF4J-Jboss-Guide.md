#  Quick guide for SLF4J use with Jboss internal logging framework.
## Introduction
The Simple Logging Facade for Java (SLF4J) serves as a simple facade or abstraction for various logging frameworks (e.g. java.util.logging, logback, log4j) allowing the end user to plug in the desired logging framework at deployment time.
This means that in you project you can use SLF4J's logging functionalities without worrying about the underlying logging framework. What logging framework will be used is completely independent from what you write in your code and how you log.

In this guide we'll cover how to use SLF4J and let the **JBoss** internal logging framework do the rest of the work.
## 1) pom.xml configuration
First of all, **hierarchy**.  JBoss has its own and SLF4J is not in the first position:
1.  JBoss LogManager (mainly used only inside the WildFly app server)
    
2.  Log4j 2
    
3.  Log4j 1
    
4.  Slf4j
    
5.  JDK logging

Note the order above, it is important. By default JBoss Logging will search the ClassLoader for the availability of classes that indicate one of the above "providers" being available. It does this in the order defined above. So, for example, if you have both JBoss LogManager and Slf4j available your classpath then JBoss LogManager will be used as it has the "higher precedence".

So, in order to use SLF4J, the better option is to delete/comment any dependence to other logging frameworks from the **pom.xml** file, and add the dependency to the **SL4J API**.

	<dependency>
	    <groupId>org.slf4j</groupId>
		<artifactId>slf4j-api</artifactId>
		<version>1.7.36</version>
	</dependency>
Please check https://mvnrepository.com/artifact/org.slf4j/slf4j-api to get the latest (stable or not, your choice) version.

This is more than enough to start using SLF4J  in your code.
## 2) Using SLF4J in your code
At this point, you project is ready to use SLF4J logging, so let's take a look at a simple example:

	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;

	public class Slf4jTest {
    private static final Logger logger = LoggerFactory.getLogger(Slf4jTest.class);

	    public Slf4jTest(){
	        logger.info( "Hello World!" );
	    }
	}

In this example the static method **getLogger** of the **LoggerFactory** returns a **Logger** used to log in the constructor method.
How and where will it log? It depends on how the underlying logging framework is configured. If the application is deployed on a JBoss server, it all depends on how the server's logging subsystem is configured.
## 3) JBoss logging subsystem:
### Default logging profile:
By default, JBoss logs the same on the console and on a **server.log** file that you will find in the **standalone/log** folder inside you JBoss directory. This is not ideal: if you have more than one application deployed on the same server, you will read all logs in the same file, mixed with logs from other applications running on the same server.
### Custom logging profile:
Imagine having more applications deployed on your server: you don't want to have all the logs mixed in one file. It is possible to define **mutiple logging profiles** with different behaviors, that can be used by different deployed applications to have different logging behaviors. For example, you can have all the logging files related to your application in a particular folder, separated from all the other applications.
In order to specify what specific logging profile you want to use, you will need the following line inside your **MANIFEST.MF** file:

	Logging-Profile:example_custom_profile

This file is generated at build time. In order to have this line in your MANIFEST.MF file, you need to specify it inside the **pom.xml** file, inside the build declaration. For example, if you're building with maven-war-plugin you need something like this:

	<build>
		<plugins>
			<plugin>
				<artifactId>maven-war-plugin</artifactId>
				<version>3.0.0</version>
				<configuration>
					<warSourceDirectory>src/main/webapp</warSourceDirectory>
					<archive>
						<manifestEntries>
							<Logging-Profile>example_custom_profile</Logging-Profile>
						</manifestEntries>
					</archive>
				</configuration>
			</plugin>
		</plugins>
	</build>

With this setup, your deployed application will look for your specified logging profile and use it. If no such profile is found, the default logging profile will be used.
How and where the application will log, will now depend only on how the logging profile is configured. 
The positive thing about this is that it will be possible for the server administrator to decide the configuration of the logging profile, decide where to write, what file rolling policies to use and so on. The only thing that needs to be in coordination with who is providing the deployed application is the name of the logging profile.
## Defining a custom logging profile on JBoss
All that is left is defining a custom logging profile for your application to use. Let's do this on the administration console, which, by default, can be found at http://127.0.0.1:9990 when JBoss is running.
Open **CONFIGURATION-->Subsystems** and search for **logging**. Select **Logging profiles** and then click on the **+** in a circle (Add logging profile) to add a new profile. Provide a name and press **Add**. Then select on the newly created profile and press view.
You should now see the profile configuration page. What you need to do now is defining on or more handlers to take care of your logs.
Once you've defined one or more handlers, you can now go back and edit the Root Logger for this logging profile (or create a specific logger category), adding the handlers you just created to it. At this point, your application should log using the specific behavior defined by your custom logging profile.

## References
* [Jboss Logging Guide](https://docs.jboss.org/hibernate/orm/current/topical/html_single/logging/Logging.html)
* [SLF4J Website](https://www.slf4j.org/)