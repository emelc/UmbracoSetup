# Logging

## Log4Net: 

Update the following file: `~/config/log4net.config`. 

This adds range filters to prevent debug / info / warn messages being logged on production.

~~~xml
<?xml version="1.0"?>
<log4net>
  
  <root>
    <priority value="Info"/>
    <appender-ref ref="AsynchronousLog4NetAppender" />
  </root>

  <appender name="rollingFile" type="log4net.Appender.RollingFileAppender">
    <file type="log4net.Util.PatternString" value="App_Data\Logs\UmbracoTraceLog.%property{log4net:HostName}.txt" />
    <lockingModel type="log4net.Appender.FileAppender+MinimalLock" />
    <appendToFile value="true" />
    <rollingStyle value="Date" />
    <maximumFileSize value="5MB" />
    <layout type="log4net.Layout.PatternLayout">
      <conversionPattern value=" %date [P%property{processId}/D%property{appDomainId}/T%thread] %-5level %logger - %message%newline" />
    </layout>
    <encoding value="utf-8" />
    <filter type="log4net.Filter.LevelRangeFilter">
      <param name="LevelMin" value="ERROR"/>
      <param name="LevelMax" value="ERROR"/>
    </filter>
  </appender>
  <appender name="AsynchronousLog4NetAppender" type="Log4Net.Async.ParallelForwardingAppender,Log4Net.Async">
    <appender-ref ref="rollingFile" />
  </appender>

  <logger name="NHibernate">
    <level value="WARN" />
  </logger>
</log4net>
~~~