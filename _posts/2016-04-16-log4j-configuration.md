---
layout: post
title: Learning log4j configuration once and for all
enabledisqus: true
---

Logging is an essential part of programming, and in Java, you are bound to run into slf4j and log4j some time. Even if you have no idea what they actually do, you are probably using them already. And you may have even used some time to learn how to configure log4j enough to modify the tiny part for your projects need. Maybe you went the extra mile and changed the log config for tests and even started using placeholders. You realized you used more time than you actually should because of many trials and failures, cursing each time that damned configuration changes weren't picked up, but finally you got it to work just the way you like it.

Then a year pasts. You start a new project. The logging doesn't seem so good. You don't remember anything from last time you entered the realm of logging frameworks. The stuff start to come back to you as you do your research. You remember all the pain while you are reliving it. You try and fail, and BAAM! you have again used 2 days working out logging. This post is to remind my self, and hopefully others, of this pain the next time I work with logging so I hopefully do not use full days just to fix logging issues!

**NOTE: This text is valid for log4j version 1.x.x.**

### What is difference between SLF4j and Log4j?
This is not so important in daily programming, but SLF4j (Simple Logging Facade) is a logging facade/abstraction/standard that can be implemented by a logging framework, which then enables easy replacement of the framework. java.util.logging, logback and log4j support SLF4j


### Log4j main components
The main components in log4j are _loggers_, _appenders_ and _layout_. Logger use appenders, appenders use layout. Logger defines what classes/packages to log and the log level. Append defines where to write the log, like file or console. Layout defines how the log messages are formatted. Many loggers can use same appender, and multiple appenders can use same layout. This is fine. The tricky part is that loggers inherit from each other based on the package name of what to log. And, in addition you have the root logger that every other logger inherits from.  If not taken care, the inherit mechanism can make the same log message appear multiple times in your log files! An example: logger for package a.b.c.d use appender x, and logger a.b.c also use appender x, then each log message from a.b.c.d classes will appear two times in x, because logger a.b.c inherits the appender of a.b.c.d. There is a flag, called additivity, that can prevent this from happening, but when you are looking are a log4j configuration after a long time, this is easy to forget.

### Log4j can be configured using Java properties file!
I actually used some time to figure this out! In some code I was reading, I saw log4j.properties file with a lot of constants. I assumed that there probably was a log4j.xml file somewhere in my application dependency which was using the property file to fill out its placeholders, but I couldn't find it. Eventually, I understood that log4j.properties file alone did the magic, but I struggeled to find the proper syntax, except from examples that compared xml configuration with property file configuration. The benefit of the xml configuration file is that it follows as schema so that you know what attributes are valid, which helps when writing configuration after a long time.

### Log4j can use property placeholders (type `${property}`), but you must be cautious!
Log4j resolves `${}` typed properties placeholders when reading the configuration from system properties. The thing is, they must have been set before log4j reads the configuration file or else they will not be resolved. Since loggers code are typically referenced using static class variables (using getLogger()), they are resolved when the class is loaded by the jvm. This means that log4j reads its configuration when the class is loaded. If you are setting the environment variables in a `@before` or `@beforeclass` method of your junit test, then you are too late! Log4j would already have read the configuration and tried to resolve the property placeholders getting nothing! There are several solution to this, but an obvious one is to use the -D option of java command linewhen running the application to send environment properties from command line. But when running test, you may not want or have the opportunity of using the -D option. Then, maybe you should use one dedicated log4j configuration file for production where the property placeholder are used, and another one for test, where you hardcode the placeholder resolution. NOTE: log4j version 2 has support for more property placeholder magic, which help solve some of the challenges stated [here](http://logging.apache.org/log4j/2.x/manual/configuration.html)

### Using log4j.xml and log4j.properties together
If you use log4j.xml in production (main/resource/log4j.xml), and use a log4j.properties file in test (test/resource/log4j.properties), then the log4j.properties will always be ignored, also when running tests. To overide the xml in test, you must use another xml. Xml has precedense over .properties files!

### You should not include log4j configuration file in you jars/application packages
If other applications are using you jar, and the log4j configuration is included in the jar, then your configuration may conflict with the user application. You should always exclude the configuration when packaging, for example using mavens jar plugin.
