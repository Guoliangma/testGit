# Log4jdbc 配置清单

```xml
	<dependency>
		<groupId>org.bgee.log4jdbc-log4j2</groupId>
		<artifactId>log4jdbc-log4j2-jdbc4</artifactId>
		<version>1.16</version>
	</dependency>
	
	配置真实驱动：log4jdbc.log4j2.properties
	
	config.properties dataSource.driverClass=net.sf.log4jdbc.sql.jdbcapi.DriverSpy
	
	url修改 jdbc\:log4jdbc\:ArteryBase\://172.25.16.68\:5432/JXGLPT_NEW?currentSchema\=db_gjyjda&Charset\=utf8
	
	原始url：jdbc\:ArteryBase\://172.25.16.68\:5432/JXGLPT_NEW?currentSchema\=db_gjyjda&Charset\=utf8
	
	logback.xml中处理对应日志级别
	
<logger name="jdbc.sqlonly" level="INFO" additivity="false">
	<appender-ref ref="JDBC" />
</logger>
<logger name="jdbc.sqltiming" level="INFO" additivity="false">
	<appender-ref ref="JDBC" />
</logger>
<logger name="jdbc.resultsettable" level="warn" additivity="false">
	<appender-ref ref="JDBC" />
</logger>
<logger name="jdbc.resultset" level="warn" additivity="false">
	<appender-ref ref="JDBC" />
</logger>
<logger name="jdbc.connection" level="warn" additivity="false">
	<appender-ref ref="JDBC" />
</logger>
<logger name="jdbc.audit" level="warn" additivity="false">
	<appender-ref ref="JDBC" />
</logger>
	
<appender name="JDBC" class="ch.qos.logback.core.rolling.RollingFileAppender">
	<File>D:/artery/gjyjda-jdbc.log</File>
	<encoder>
		<pattern>%d{yyyy-MM-dd HH:mm:ss:SSS , GMT+8} [%c:%L]-[%p] %m%n
		</pattern>
	</encoder>
	<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
		<fileNamePattern>D:/artery/gjyjda_jdbc_${IP}_${PORT}.%d.log</fileNamePattern>
	</rollingPolicy>
</appender>
```