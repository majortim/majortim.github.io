---
layout: post
title:  "JavaMailSender로 메일 보내기"
description: "Spring Framework와 Jakarta Mail을 이용한 메일 보내기"
date:   2022-04-22 22:55:00 +0900
categories: Spring
excerpt_separator: <!--more-->
---


스프링은 유용한 메일 전송 API를 제공합니다. org.springframework.mail 패키지 아래에 있는 API 중 JavaMailSender를 이용해 메일 전송하는 방법을 소개합니다.

<!--more-->

우선 다음과 같은 의존성이 포함되어 있어야 합니다.

```xml
<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-context-support</artifactId>
		<version>5.3.19</version>
</dependency>
<dependency>
		<groupId>com.sun.mail</groupId>
		<artifactId>jakarta.mail</artifactId>
		<version>1.6.7</version>
</dependency>
```

Spring 5 버전대에선 Jakarta Mail의 네임스페이스가 javax.mail 이기 때문에 Jakarta Mail은 2 버전대가 아니라 1.6 버전대를 이용합니다.

다음으로 프로퍼티 파일입니다.

```
mail.smtp.host=smtp.naver.com
mail.smtp.port=465
mail.transport.protocol=smtp
```

사용하는 메일 서비스의 설정에 따라 포트가 달라질 수 있습니다. 보통 465 또는 587 포트를 이용합니다. 네이버 메일 서비스를 사용하는 경우 SMTP 사용 설정을 해야합니다. [참조](https://help.naver.com/support/service/main.help?serviceNo=2342&categoryNo=12648)

그리고 Configuration 클래스에 JavaMailSender 빈을 다음과 같이 정의합니다.

```java
package org.example.mail.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.JavaMailSenderImpl;

import java.util.Properties;

@Configuration
@PropertySource("classpath:mail.properties")
public class AppConfig {
    @Value("${mail.transport.protocol}")
    private String protocol;

    @Value("${mail.smtp.host}")
    private String host;
    @Value("${mail.smtp.port}")
    private int port;
    @Value("${mail.username}")
    private String username;
    @Value("${mail.password}")
    private String password;

    @Bean
    public JavaMailSender javaMailSender() {
        JavaMailSenderImpl sender = new JavaMailSenderImpl();

        sender.setProtocol(protocol);
        sender.setHost(host);
        sender.setPort(port);
        sender.setUsername(username);
        sender.setPassword(password);

        Properties props = sender.getJavaMailProperties();
				props.put("mail.smtp.starttls.enable", "true");
        props.put("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory");

//        props.put("mail.debug", "true");

        return sender;
    }
}
```

@PropertySource 애노테이션을 이용해 사용할 프로퍼티 경로를 지정해줍니다. host 와 port 속성은 이 지정한 프로퍼티 파일에서 가져옵니다. 프로퍼티 파일을 버전 관리 대상에 포함했기 때문에 username과 password는 프로퍼티 파일에 넣지 않고 프로그램 실행시 시스템 프로퍼티를 이용하도록 했습니다(VM 옵션에 다음과 같은 형식으로 전달: -Dmail.username=사용자계정 -Dmail.password=비밀번호).

TLS 전송을 위해 mail.smtp.starttls.enable 속성과 mail.smtp.socketFactory.class 속성을 추가했습니다. 위의 속성은 Jakarta Mail API 표준에 정의된 속성이 아니라 프로바이더에 특화된 속성입니다.

```jsx
package org.example.mail.main;

import org.example.mail.config.AppConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.GenericApplicationContext;
import org.springframework.core.env.Environment;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;

public class Main {
    public static void main(String[] args) {
        try (GenericApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class)) {
            
            JavaMailSender sender = context.getBean(JavaMailSender.class);

            Environment environment = context.getEnvironment();
            String from = environment.getRequiredProperty("mail.from");
            String to = environment.getRequiredProperty("mail.to");

            SimpleMailMessage message = new SimpleMailMessage();
            message.setFrom(from);
            message.setTo(to);
            message.setSubject("메일 테스트");
            message.setText("테스트");

            sender.send(message);
        }
    }
}
```

메인 메서드가 정의된 클래스입니다. username, password와 마찬가지로 발송자(from), 수신자(to) 메일주소도 VM 옵션을 이용해 지정한 시스템 프로퍼티로부터 가져옵니다. 단순한 텍스트 메시지를 보낼 땐 위와 같이 SimpleMailMessage 클래스를 이용하면 됩니다.

```java
17:55:08.808 [main] INFO  javax.mail - Jakarta Mail version 1.6.7
17:55:08.808 [main] INFO  javax.mail - successfully loaded resource: /META-INF/javamail.default.providers
17:55:08.824 [main] INFO  javax.mail - Tables of loaded providers
17:55:08.824 [main] INFO  javax.mail - Providers Listed By Class Name: {com.sun.mail.smtp.SMTPTransport=javax.mail.Provider[TRANSPORT,smtp,com.sun.mail.smtp.SMTPTransport,Oracle], com.sun.mail.imap.IMAPSSLStore=javax.mail.Provider[STORE,imaps,com.sun.mail.imap.IMAPSSLStore,Oracle], com.sun.mail.pop3.POP3Store=javax.mail.Provider[STORE,pop3,com.sun.mail.pop3.POP3Store,Oracle], com.sun.mail.smtp.SMTPSSLTransport=javax.mail.Provider[TRANSPORT,smtps,com.sun.mail.smtp.SMTPSSLTransport,Oracle], com.sun.mail.imap.IMAPStore=javax.mail.Provider[STORE,imap,com.sun.mail.imap.IMAPStore,Oracle], com.sun.mail.pop3.POP3SSLStore=javax.mail.Provider[STORE,pop3s,com.sun.mail.pop3.POP3SSLStore,Oracle]}
17:55:08.824 [main] INFO  javax.mail - Providers Listed By Protocol: {imap=javax.mail.Provider[STORE,imap,com.sun.mail.imap.IMAPStore,Oracle], smtp=javax.mail.Provider[TRANSPORT,smtp,com.sun.mail.smtp.SMTPTransport,Oracle], pop3=javax.mail.Provider[STORE,pop3,com.sun.mail.pop3.POP3Store,Oracle], imaps=javax.mail.Provider[STORE,imaps,com.sun.mail.imap.IMAPSSLStore,Oracle], smtps=javax.mail.Provider[TRANSPORT,smtps,com.sun.mail.smtp.SMTPSSLTransport,Oracle], pop3s=javax.mail.Provider[STORE,pop3s,com.sun.mail.pop3.POP3SSLStore,Oracle]}
17:55:08.824 [main] INFO  javax.mail - successfully loaded resource: /META-INF/javamail.default.address.map
17:55:08.824 [main] DEBUG javax.mail - getProvider() returning javax.mail.Provider[TRANSPORT,smtp,com.sun.mail.smtp.SMTPTransport,Oracle]
17:55:08.839 [main] DEBUG com.sun.mail.smtp - useEhlo true, useAuth false
17:55:08.839 [main] DEBUG com.sun.mail.smtp - trying to connect to host "smtp.naver.com", port 465, isSSL false
17:55:08.839 [main] DEBUG com.sun.mail.util.socket - getSocket, host smtp.naver.com, port 465, prefix mail.smtp, useSSL false
17:55:08.980 [main] TRACE com.sun.mail.util.socket - create socket: prefix mail.smtp, localaddr null, localport 0, host smtp.naver.com, port 465, connection timeout -1, timeout -1, socket factory sun.security.ssl.SSLSocketFactoryImpl@20765ed5, useSSL false
17:55:08.980 [main] TRACE com.sun.mail.util.socket - connecting...
17:55:08.996 [main] TRACE com.sun.mail.util.socket - success!
17:55:09.011 [main] DEBUG com.sun.mail.util.socket - SSL enabled protocols before [TLSv1.3, TLSv1.2, TLSv1.1, TLSv1]
17:55:09.011 [main] DEBUG com.sun.mail.util.socket - SSL enabled protocols after [TLSv1.3, TLSv1.2, TLSv1.1, TLSv1]
17:55:09.011 [main] DEBUG com.sun.mail.util.socket - SSL enabled ciphers after [TLS_AES_256_GCM_SHA384, TLS_AES_128_GCM_SHA256, TLS_CHACHA20_POLY1305_SHA256, TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384, TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256, TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256, TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384, TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256, TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256, TLS_DHE_RSA_WITH_AES_256_GCM_SHA384, TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256, TLS_DHE_DSS_WITH_AES_256_GCM_SHA384, TLS_DHE_RSA_WITH_AES_128_GCM_SHA256, TLS_DHE_DSS_WITH_AES_128_GCM_SHA256, TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384, TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384, TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256, TLS_DHE_RSA_WITH_AES_256_CBC_SHA256, TLS_DHE_DSS_WITH_AES_256_CBC_SHA256, TLS_DHE_RSA_WITH_AES_128_CBC_SHA256, TLS_DHE_DSS_WITH_AES_128_CBC_SHA256, TLS_ECDH_ECDSA_WITH_AES_256_GCM_SHA384, TLS_ECDH_RSA_WITH_AES_256_GCM_SHA384, TLS_ECDH_ECDSA_WITH_AES_128_GCM_SHA256, TLS_ECDH_RSA_WITH_AES_128_GCM_SHA256, TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA384, TLS_ECDH_RSA_WITH_AES_256_CBC_SHA384, TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256, TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256, TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA, TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA, TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_RSA_WITH_AES_256_CBC_SHA, TLS_DHE_DSS_WITH_AES_256_CBC_SHA, TLS_DHE_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_DSS_WITH_AES_128_CBC_SHA, TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA, TLS_ECDH_RSA_WITH_AES_256_CBC_SHA, TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA, TLS_ECDH_RSA_WITH_AES_128_CBC_SHA, TLS_RSA_WITH_AES_256_GCM_SHA384, TLS_RSA_WITH_AES_128_GCM_SHA256, TLS_RSA_WITH_AES_256_CBC_SHA256, TLS_RSA_WITH_AES_128_CBC_SHA256, TLS_RSA_WITH_AES_256_CBC_SHA, TLS_RSA_WITH_AES_128_CBC_SHA, TLS_EMPTY_RENEGOTIATION_INFO_SCSV]
17:55:09.183 [main] TRACE com.sun.mail.smtp.protocol - 220 smtp.naver.com ESMTP  - nsmtp
17:55:09.183 [main] DEBUG com.sun.mail.smtp - connected to host "smtp.naver.com", port: 465
17:55:09.183 [main] TRACE com.sun.mail.smtp.protocol - EHLO IP
17:55:09.183 [main] TRACE com.sun.mail.smtp.protocol - 250-smtp.naver.com Pleased to meet you
17:55:09.183 [main] TRACE com.sun.mail.smtp.protocol - 250-SIZE 20971520
17:55:09.183 [main] TRACE com.sun.mail.smtp.protocol - 250-8BITMIME
17:55:09.183 [main] TRACE com.sun.mail.smtp.protocol - 250-PIPELINING
17:55:09.183 [main] TRACE com.sun.mail.smtp.protocol - 250-AUTH PLAIN LOGIN
17:55:09.183 [main] TRACE com.sun.mail.smtp.protocol - 250 ENHANCEDSTATUSCODES
17:55:09.183 [main] DEBUG com.sun.mail.smtp - Found extension "SIZE", arg "20971520"
17:55:09.183 [main] DEBUG com.sun.mail.smtp - Found extension "8BITMIME", arg ""
17:55:09.183 [main] DEBUG com.sun.mail.smtp - Found extension "PIPELINING", arg ""
17:55:09.183 [main] DEBUG com.sun.mail.smtp - Found extension "AUTH", arg "PLAIN LOGIN"
17:55:09.183 [main] DEBUG com.sun.mail.smtp - Found extension "ENHANCEDSTATUSCODES", arg ""
17:55:09.183 [main] DEBUG com.sun.mail.smtp - STARTTLS requested but already using SSL
17:55:09.183 [main] DEBUG com.sun.mail.smtp - protocolConnect login, host=smtp.naver.com, user=발송자, password=<non-null>
17:55:09.183 [main] DEBUG com.sun.mail.smtp - Attempt to authenticate using mechanisms: LOGIN PLAIN DIGEST-MD5 NTLM XOAUTH2 
17:55:09.183 [main] DEBUG com.sun.mail.smtp - Using mechanism LOGIN
17:55:09.183 [main] DEBUG com.sun.mail.smtp - AUTH LOGIN command trace suppressed
17:55:09.251 [main] DEBUG com.sun.mail.smtp - AUTH LOGIN succeeded
17:55:09.261 [main] DEBUG com.sun.mail.smtp - use8bit false
17:55:09.261 [main] TRACE com.sun.mail.smtp.protocol - MAIL FROM:<발송자@naver.com>
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - 250 2.1.0 OK  - nsmtp
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - RCPT TO:<수신자@gmail.com>
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - 250 2.1.5 OK  - nsmtp
17:55:09.277 [main] DEBUG com.sun.mail.smtp - Verified Addresses
17:55:09.277 [main] DEBUG com.sun.mail.smtp -   수신자@gmail.com
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - DATA
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - 354 Go ahead  - nsmtp
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - Date: Fri, 22 Apr 2022 17:55:09 +0900 (KST)
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - From: 발송자@naver.com
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - To: 수신자@gmail.com
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - Message-ID: <>
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - Subject: =?UTF-8?B?66mU7J28IO2FjOyKpO2KuA==?=
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - MIME-Version: 1.0
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - Content-Type: text/plain; charset=UTF-8
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - Content-Transfer-Encoding: base64
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - 
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - 7YWM7Iqk7Yq4
17:55:09.277 [main] TRACE com.sun.mail.smtp.protocol - .
17:55:09.568 [main] TRACE com.sun.mail.smtp.protocol - 250 2.0.0 OK  - nsmtp
17:55:09.569 [main] DEBUG com.sun.mail.smtp - message successfully delivered to mail server
17:55:09.569 [main] TRACE com.sun.mail.smtp.protocol - QUIT
17:55:09.572 [main] TRACE com.sun.mail.smtp.protocol - 221 2.0.0 Closing connection  - nsmtp
```

실행화면 로그입니다. 메시지를 성공적으로 발송했습니다.  JUL 설정을 추가했기 때문에 mail.debug 속성을 true로 지정했을 때보다 더 상세한 로그가 나옵니다.

## JUL 설정(옵션)

```java
package org.example.mail.log;

import org.slf4j.bridge.SLF4JBridgeHandler;

import java.util.logging.Level;
import java.util.logging.LogManager;
import java.util.logging.Logger;

public class JavaUtilLoggingSetup {
    public static void setup() {
        SLF4JBridgeHandler.removeHandlersForRootLogger();  // (since SLF4J 1.6.5)
        SLF4JBridgeHandler.install();

        LogManager manager = java.util.logging.LogManager.getLogManager();
        Logger javaxMail = Logger.getLogger("javax.mail");
        Logger comSunMail = Logger.getLogger("com.sun.mail");
        javaxMail.setLevel(Level.FINEST);
        comSunMail.setLevel(Level.FINEST);
        manager.addLogger(javaxMail);
        manager.addLogger(comSunMail);
    }
}
```

Jakarta Mail은 JUL을 이용하기 때문에 jul-to-slf4j bridge를 이용해 JUL 처리를 SLF4J에 위임하는 설정을 넣습니다. FINEST 레벨은 SLF4J의 TRACE 레벨에 해당합니다. 메인 메서드에서 위의 static 메서드를 호출하면 설정이 반영됩니다.

지금까지 Spring 기반 애플리케이션에서 JavaMailSender를 이용한 메일 발송에 대해 알아봤습니다. 파일 첨부, [메일 템플릿](https://www.baeldung.com/spring-email-templates) 이용 등은 SimpleMailMessage 가 아니라 [MimeMessage](https://docs.spring.io/spring-framework/docs/5.3.x/javadoc-api/org/springframework/mail/javamail/MimeMessageHelper.html) 를 이용하시면 됩니다.

* Gmail 발송이 안 되는 경우 계정이 2단계 인증을 사용하고 있는지 확인합니다. 2단계 인증을 사용하는 경우 전용 [앱 비밀번호](https://support.google.com/mail/answer/185833?hl=ko)를 생성해야 합니다.

# 참조

[https://docs.spring.io/spring-framework/docs/5.3.x/reference/html/integration.html#mail](https://docs.spring.io/spring-framework/docs/5.3.x/reference/html/integration.html#mail)

[https://eclipse-ee4j.github.io/mail](https://eclipse-ee4j.github.io/mail)