---
layout: post
title: Project gallendar - Trouble Shooting
comments: true
tags: project
---

<h4>오류 내용</h4>

**SMTP ERROR**

`Failed message 1: com.sun.mail.smtp.SMTPSendFailedException: 530 5.7.0 Must issue`

gardle의 yml 파일 setting의 오류로써 gradle을 사용하시는 분들은 아래 부분을 참고하여 yml setting을 해주셔야 오류가 나지 않습니다.

```java
  mail:
    properties:
      mail:
        smtp:
          starttls:
            enable: true
          auth: true
    host: 
    username: 
    password: 
    port: 587
  redis:
    host: 127.0.0.1
    port: 6379

```

**젠킨스 build error**

`echo "Done"
FATAL: Unable to produce a script file
java.nio.charset.UnmappableCharacterException: Input length = 1
	at java.base/java.nio.charset.CoderResult.throwException(CoderResult.java:275)
	at java.base/sun.nio.cs.StreamEncoder.implWrite(StreamEncoder.java:306)
	at java.base/sun.nio.cs.StreamEncoder.implWrite(StreamEncoder.java:281)
	at java.base/sun.nio.cs.StreamEncoder.write(StreamEncoder.java:125)
	at java.base/java.io.OutputStreamWriter.write(OutputStreamWriter.java:208)
	at java.base/java.io.BufferedWriter.flushBuffer(BufferedWriter.java:120)
	at java.base/java.io.BufferedWriter.close(BufferedWriter.java:268)
	at hudson.FilePath$CreateTextTempFile.invoke(FilePath.java:1652)
	at hudson.FilePath$CreateTextTempFile.invoke(FilePath.java:1622)
	at hudson.FilePath.act(FilePath.java:1192)
	at hudson.FilePath.act(FilePath.java:1175)
	at hudson.FilePath.createTextTempFile(FilePath.java:1616)
Caused: java.io.IOException: Failed to create a temp file on /var/lib/jenkins/workspace/gallendar
	at hudson.FilePath.createTextTempFile(FilePath.java:1618)
	at hudson.tasks.CommandInterpreter.createScriptFile(CommandInterpreter.java:202)
	at hudson.tasks.CommandInterpreter.perform(CommandInterpreter.java:120)
	at hudson.tasks.CommandInterpreter.perform(CommandInterpreter.java:92)
	at hudson.plugins.postbuildtask.PostbuildTask.perform(PostbuildTask.java:123)
	at hudson.tasks.BuildStepMonitor$1.perform(BuildStepMonitor.java:20)
	at hudson.model.AbstractBuild$AbstractBuildExecution.perform(AbstractBuild.java:816)
	at hudson.model.AbstractBuild$AbstractBuildExecution.performAllBuildSteps(AbstractBuild.java:765)
	at hudson.model.Build$BuildExecution.post2(Build.java:179)
	at hudson.model.AbstractBuild$AbstractBuildExecution.post(AbstractBuild.java:709)
	at hudson.model.Run.execute(Run.java:1924)
	at hudson.model.FreeStyleBuild.run(FreeStyleBuild.java:44)
	at hudson.model.ResourceController.execute(ResourceController.java:107)
	at hudson.model.Executor.run(Executor.java:449)
POST BUILD TASK : FAILURE
END OF POST BUILD TASK : 0
Finished: SUCCESS`

yml 및 setting 파일의 주석 부분에 UTF-8 한글문자가 있어서 발생한 오류이다. 이는 프로젝트 셋팅을 UTF-8을 명시해줄 수 있는 라인이 필요로 하다.

**AWS 배포 에러 로그**

`SdkClientException: The requested metadata is not found at`
` Unable to retrieve the requested metadata (/latest/user-data/)`

yml의 버킷이름과 시크릿 키를 확인해야 한다.

**WebSecurityConfigurerAdapter deprecated**

`[spring security] Error creating bean with name 에러 / 순환 참조 에러`

5.7버전부터 WebSecurityConfigurerAdapter 가 deprecated 되었다. 기존에는 WebSecurityConfigurerAdapter를 extends 해서 메소드를 오버라이딩 하여 구현하였다. 최근 공식문서를 참고해보자.

```
Deprecated.
Use a SecurityFilterChain Bean to configure HttpSecurity or a WebSecurityCustomizer Bean to configure WebSecurity
````
간단하게 보면 SecurityFilterChain을 Bean으로 등록해서 사용하라는 메시지를 남겨주고 있습니다. [공식문서](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter) 에 의하여 `@Bean
public SecurityFilterChain configure(HttpSecurity http) throws Exception` 이런식으로 Bean으로 등록하여 최신 Security에 맞춰 사용하면 오류를 제거할 수 있다.
