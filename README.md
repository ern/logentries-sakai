# Logentries and Raven Java (for Sentry) for Sakai

This project re-packages raven-java and the logentries-appendar and all dependencies so that they can be deployed 
into Sakai's classloaders, this allows you to send log4j events to sentry.

## Deployment

Copy the artifact into tomcat's `common/lib` folder so that it is in the same classloader as log4j.jar, if you are
running a copy of Sakai before 10.x you may get problems with tools supplying thier own copy of log4j.jar. Then
you need to configure log4j to use the raven logger, placing a log4j.properties file in `sakai.home` means it will
get read at startup and watched for changes. A simple example of this is:

    # Define the loggers
    log4j.rootLogger=WARN, console, sentry, le
    
    # Sentry setup
    log4j.appender.sentry.dsn=https://publicKey:secretKey@host:port/1?options
    log4j.appender.sentry=net.kencochrane.raven.log4j.SentryAppender
    log4j.appender.sentry.Threshold=ERROR

    # Logentries setup
    log4j.appender.le=com.logentries.log4j.LogentriesAppender
    log4j.appender.le.layout=org.apache.log4j.PatternLayout
    log4j.appender.le.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss ZZZ} %-5p: %F:%L  %m
    log4j.appender.le.Token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    log4j.appender.le.Threshold=INFO

    # Console setup
    log4j.appender.console=org.apache.log4j.ConsoleAppender
    log4j.appender.console.layout=org.apache.log4j.PatternLayout
    log4j.appender.console.layout.ConversionPattern=%d{ISO8601} %5p %t %c - %m%n
    log4j.appender.console.Encoding=UTF-8
    
    # Application logging options
    log4j.logger.org.apache=INFO
    log4j.logger.org.sakaiproject=INFO
    log4j.logger.uk.ac.cam.caret.rwiki=INFO
    log4j.logger.org.theospi=INFO
    log4j.logger.MySQL=INFO
    log4j.logger.uk.ac.ox.oucs=INFO
    log4j.logger.net.sf.snmpadaptor4j=INFO
    
To avoid putting secrets in configuration files you can use the `SENTRY_DSN` envrionment variable to configure the 
raven-java log4j integration.

## Custom hostname

Sometimes the way raven-java looks up the hostname isn't correct, by default it will use the name of the IP used
on the default interface. In some setups this isn't correct, for example when running inside docker containers with
custom networking, if you wish it to just use the `HOSTNAME` environment variable then you can set the raven factory
to be `org.sakaiproject.sentry.DockerRavenFactory` and it will use that instead, if using log4j.properties then the
configuration to enable this is:

    log4j.appender.sentry.ravenFactory=org.sakaiproject.sentry.DockerRavenFactory
    
## Credit

Original author of raven-java for Sakai: @buckett
