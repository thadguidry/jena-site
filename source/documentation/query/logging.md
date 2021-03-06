---
title: ARQ - Logging
---

ARQ uses
[SLF4j](http://slf4j.org/)
as the API and ships with
[Log4J](http://logging.apache.org/log4j/) as a deployment
system.  You can use Java 1.4 logging instead.

ARQ does not output any logging messages at level INFO in normal
operation. The code uses level TRACE and DEBUG.  Running with
logging set to an application at INFO will cause no output in
normal operation. Output below INFO can be very verbose and is
intended mainly to help debug ARQ. WARN and FATAL messages are only
used when something is wrong.

The root of all the loggers is `org.apache.jena`.
`org.apache.jena.query` is the application API. 
`org.apache.jena.sparql` is the implementation and extensions
points.

If using in Tomcat, or other system that provides complex class
loading arrangements, be careful about loading from jars in both
the web application and the system directories as this can cause
separate logging systems to be created (this may not matter).

The ARQ and RIOT command line utilities look for a file
"log4j.properties" in the current directory to control logging during
command execution.

Logger Names | Name | Constant | Logger | Use
------------ | ---- | -------- | ------ | ---
`org.apache.jena.arq.info` | `ARQ.logInfoName` | `ARQ.getLoggerInfo()` | General information
`org.apache.jena.arq.exec` | `ARQ.logExecName` | `ARQ.getLoggerExec()` | Execution information

There is some code to cover simple situations. See the [log4j
documentation](http://logging.apache.org/log4j/1.2/manual.html)
for full details of log4j.

The reading of `log4j.properties` from the current directory is achieved
by a call to `org.apache.jena.atlas.logging.Log.setlog4j()`.

Example log4j.properties file:

    log4j.rootLogger=INFO, stdlog

    log4j.appender.stdlog=org.apache.log4j.ConsoleAppender
    ## log4j.appender.stdlog.target=System.err
    log4j.appender.stdlog.layout=org.apache.log4j.PatternLayout
    log4j.appender.stdlog.layout.ConversionPattern=%d{HH:mm:ss} %-5p %-25c{1} :: %m%n

    # Execution logging
    log4j.logger.org.apache.jena.arq.info=INFO
    log4j.logger.org.apache.jena.arq.exec=INFO

    # Other
    log4j.logger.org.apache.jena=WARN
    log4j.logger.org.apache.jena=WARN

A [Fuseki](../serving/data/index.html)
server output can include [ARQ execution logging](explain.html "ARQ/Explain").

Note: ARQ performs some direct control of logging in the test suite and
depends on log4j to compile.

## Execution Logging

ARQ can log query and update execution details globally or for an
individual operations. This adds another level of control on top of the
logger level controls.

Explanatory messages are controlled by the `Explain.InfoLevel` level in
the execution context.

The logger used is called `org.apache.jena.arq.exec`. Message are sent
at level "info". So for log4j, the following can be set in the
log4j.properties file:

    log4j.logger.org.apache.jena.arq.exec=INFO

The context setting is for key (Java constant) `ARQ.symLogExec`. To set
globally:

    ARQ.setExecutionLogging(Explain.InfoLevel.ALL) ;

and it may also be set on an individual query execution using its local
context.

     try(QueryExecution qExec = QueryExecutionFactory.create(...)) {
        qExec.getContext().set(ARQ.symLogExec, Explain.InfoLevel.ALL) ;
        ...
     }

On the command line:

     arq.query --explain --data data file --query=queryfile

The command `tdbquery` takes the same `--explain` argument.

**Information levels**

Level | Effect
----- | ------
INFO | Log each query  |
FINE | Log each query and its algebra form after optimization
ALL | Log query, algebra and every dataset access (can be expensive)
NONE | No information logged

These can be specified as string, to the command line tools, or using
the constants in `Explain.InfoLevel`.

     qExec.getContext().set(ARQ.symLogExec, Explain.InfoLevel.FINE) ;

     arq.query --set arq:logExec=FINE --data data file --query=queryfile


[ARQ documentation index](index.html)
