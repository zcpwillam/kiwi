The Linked Media Framework (LMF) is implemented as a Java Web Application that can in principle be deployed to any Java Application Server. It has been tested under Jetty 6.x and Tomcat 6.x. It can be installed using the source code or as a binary package.

# Requirements #

Hardware:
  * standard workstation (Dual-Core or similar)
  * 1GB main memory
  * about 100MB hard disk

Software:
  * Java JDK 6 or higher
  * Java Application Server (Tomcat 6.x or Jetty 6.x)
  * Database (PostgreSQL, MySQL - if not explicitly configured, an embedded H2 database will be used)



# Installation (Binary) #

The binary installation comes as a Java Web Archive (.war) file that can be deployed in any application server. The deployment procedure is as follows:


  1. Download and install the application server (Tomcat 6.0.x or Jetty 6.x) and the database you intend to use (optional, default is H2 embedded)
  1. Copy the .war file into the application server's deployment directory (Tomcat and Jetty: the webapps subdirectory)
  1. In the console where you will start the application server, set the environment variable LMF\_HOME to the directory that will be used for storing the persistent runtime data of the Linked Media Server
  1. Startup the application server and go to the deployment URL with a web browser (e.g. http://localhost:8080). The Linked Media Server will then carry out initial setup using the embedded H2 database. The default interface will also contain links to the admin area and API documentation of the Linked Media Server.
  1. (OPTIONAL) If you do not want to use H2, go to the admin interface and configure the database according to your own preferences. It is recommended that you restart the application server when you have done so.

To avoid setting the environment variable LMF\_HOME every time you open
a new terminal, you can also enter it persistently by adding the following lines to catalina.sh (Unix):

```
export LMF_HOME=<PATH-TO-HOME>
export JAVA_OPTS="-Djava.net.preferIPv4Stack=true -Xmx1024m -XX:PermSize=128m -XX:MaxPermSize=256m -XX:+UseConcMarkSweepGC -XX:+CMSPermGenSweepingEnabled -XX:+CMSClassUnloadingEnabled"
```

The latter option will give more memory to the Linked Media Server. You can even increase the value of 1024 to something higher as needed.

## Specific Settings for Jetty ##

The Linked Media Framework uses JNDI for looking up services. While most application servers have this enabled by default, Jetty needs a little bit of setup to enable JNDI functionality. The procedure is described in the [Jetty Documentation](http://docs.codehaus.org/display/JETTY/JNDI).

In short, what you need to do is to copy the plus-settings from the jetty-plus.xml file to the jetty.xml file:

```
  <Array id="plusConfig" type="java.lang.String">
    <Item>org.mortbay.jetty.webapp.WebInfConfiguration</Item>
    <Item>org.mortbay.jetty.plus.webapp.EnvConfiguration</Item>
    <Item>org.mortbay.jetty.plus.webapp.Configuration</Item>   
    <Item>org.mortbay.jetty.webapp.JettyWebXmlConfiguration</Item>
    <Item>org.mortbay.jetty.webapp.TagLibConfiguration</Item>
  </Array>
```

and then add the option

```
<Set name="configurationClasses"><Ref id="plusConfig"/></Set>
```

to the call of org.mortbay.jetty.deployer.WebAppDeployer (search jetty.xml for it).


# Installation (Source) #

It is assumed that you have already obtained the source code in some form, otherwise you will not be able to read this documentation.

Additional Requirements:
  * Mercurial (http://mercurial.selenic.com/)
  * Gradle (http://www.gradle.org)

Installation steps:
  1. Copy the file userConfig.properties.tmpl to userConfig.properties and adapt the settings to your local configuration:
    * set the "home" parameter to the LMF data and configuration directory
    * set the "host" parameter to the name of the host used for running the server
    * set the "path" parameter to the name to use for the web application (i.e. path below the host name)
    * set the database configuration as described for the binary configuration; mode can be "create" for re-creating the database or "update" for updating the database and preserving the data or "validate" for only checking whether the database conforms to the schema of the data
  1. Run "gradle configure" to configure the system according to your settings
  1. Run "gradle jettyRun" to start the LMF in Jetty

The build system can create the project configuration for various IDEs. You can create a configuration for IntelliJ IDEA by running "gradle idea" and for Eclipse by running "gradle eclipse".


## Common Database settings ##

(change name of database if different):

H2:
  * Hibernate: org.hibernate.dialect.H2Dialect
  * Driver: org.h2.Driver
  * URL:  jdbc:h2:/tmp/kiwi/db/kiwi;MVCC=true;DB\_CLOSE\_ON\_EXIT=FALSE

PostgreSQL:
  * Hibernate: org.hibernate.dialect.PostgreSQLDialect
  * Drver: org.postgresql.Driver
  * URL:  jdbc:postgresql://localhost:5432/lmf

MySQL:
  * Hibernate: org.hibernate.dialect.MySQLDialect
  * Driver: com.mysql.jdbc.Driver
  * URL:  jdbc:mysql://localhost:3306/lmf