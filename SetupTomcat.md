# Introduction #

In production environments, Apache Tomcat will be the application server of choice. This is a collection of issues arising with Tomcat installations.

# Tomcat Versions #

Believe it or not, but Tomcat can also have bugs. We recommend to use **Tomcat 6.0.33** to work with the LMF, because it is the fastest and currently most reliable version:
  * Tomcat 6.0.32 or lower contains a serious bug that does not allow to set context parameters
  * Tomcat 7.x is significantly slower and not as reliable when using CDI/Weld


# Multiple LMF Instances #

In some settings it might be desirable to set up multiple instances of the LMF in the same application server installation under different context URLs. This can be achieved by creating context definition files under

```
conf/Catalina/localhost/<appname>.xml
```

where `<appname>`.xml is a context configuration file. `<appname>` is the name of the web application, e.g. "LMF" or "MyApp". The file will contain a configuration similar to the following:

```
<Context docBase="/data/LMF/build/libs/LMF-2.0rc5-SNAPSHOT.war" unpackWAR="false" useNaming="true">
  <Parameter name="kiwi.home" value="/data/instances/lmf" override="false"/>

  <Resource name="BeanManager" auth="Container"
            type="javax.enterprise.inject.spi.BeanManager"
            factory="org.jboss.weld.resources.ManagerObjectFactory"
            />
</Context>
```

  * The docBase attribute specifies the location of the WAR file of the LMF in case it is not located in the webapps directory.
  * The value of the parameter kiwi.home provides the location of the LMF home directory (for historical reasons called kiwi.home)
  * The Resource registers a factory for creating the Java EE 6 Bean Manager. This entry will typically remain unchanged, but it is necessary for the system to work properly.