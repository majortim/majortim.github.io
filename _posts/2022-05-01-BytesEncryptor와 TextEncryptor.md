---
layout: post
title:  "BytesEncryptor와 TextEncryptor"
description: "Spring Security Crypto Module의 대칭키 암호화"
date:   2022-05-01 23:40:00 +0900
categories: Spring
excerpt_separator: <!--more-->
---

Spring Security의 Crypto 모듈은 대칭키 암호화와 키 생성, 패스워드 인코딩 기능을 제공합니다. 그 중 대칭키 암호화에 관한 인터페이스인 `BytesEncryptor`와 `TextEncryptor`를 소개합니다.

<!--more-->

# BytesEncryptor

`BytesEncryptor`를 사용하면 인코딩한 결과물의 형식이 byte[]가 됩니다. `org.springframework.security.crypto.encrypt.Encryptors` 클래스에는 `BytesEncryptor`구현체의 인스턴스를 생성하는 두 가지 메서드가 정의되어 있습니다. 

- `static BytesEncryptor stronger(java.lang.CharSequence password, java.lang.CharSequence salt)`
- `static BytesEncryptor standard(java.lang.CharSequence password, java.lang.CharSequence salt)`

두 메서드의 차이는 [싸이퍼 알고리즘의 모드](https://docs.oracle.com/en/java/javase/17/docs/specs/security/standard-names.html#cipher-algorithm-modes)입니다. stronger 메서드는 GCM을 사용하며 standard는 CBC를 사용합니다. 인터넷에서 쉽게 찾을 수 있는 예제는 대부분 CBC를 사용하지만 스프링에서 권장하는 방식은 GCM을 사용하는 stronger 메서드입니다.

```java
BytesEncryptor encryptor = Encryptors.stronger("This is password!", "0123456f");
String rawText = "This is text!";
byte[] cipherBytes = encryptor.encrypt(rawText.getBytes(StandardCharsets.UTF_8));
```

`BytesEncryptor`를 생성하는 팩토리 메서드입니다. 앞에 들어가는 password 패러미터는 암호키를 생성하기 위해 필요한 암호입니다. 뒤에는 salt인데 hex(16진수) 형식으로 인코딩해야 합니다. 만약 위에서 입력한 0123456f가 0123456g로 바뀌면 범위를 넘어가는 문자인 g 때문에 실행할 때 에러가 발생합니다. password와 salt를 올바르게 입력하면 이를 이용해 비밀키를 생성합니다.

# TextEncryptor

`TextEncryptor`를 사용하면 인코딩한 결과물의 형식이 hex로 인코딩된 스트링이 됩니다. `Encryptors` 클래스에는 `TextEncryptor`구현체의 인스턴스를 생성하는 네 가지 메서드가 정의되어 있습니다. 

- `static TextEncryptor delux​(java.lang.CharSequence password, java.lang.CharSequence salt)`
- `static TextEncryptor noOpText()`
- ~~`static TextEncryptor queryableText​(java.lang.CharSequence password, java.lang.CharSequence salt)`~~
- `static TextEncryptor text(java.lang.CharSequence password, java.lang.CharSequence salt)`

queryTableText(...) 메서드 같은 경우는 초기화 벡터(이하 iv)가 항상 일정하기 때문에(0 값이 16 바이트로 채워진) 동일한 문자열을 인코딩할 땐 몇 번을 수행해도 인코딩 결과가 똑같습니다. 안정성 문제로 deprecated 처리됐으니 사용을 자제하는 것이 좋습니다. noOpText() 메서드로 생성한 인스턴스는 암호화 자체를 하지 않기 때문에 활용도가 높지 않습니다. 남는 건 `BytesEncryptor`인터페이스처럼 두 가지 메서드밖에 없습니다. delux(...) 메서드와 text(...) 메서드인데 delux는 stronger 메서드를, text는 standard 메서드를 내부적으로 호출합니다.

```java
TextEncryptor encryptor = Encryptors.delux("This is password!", "0123456f");
String rawText = "This is text!";
String cipherText = encryptor.encrypt(rawText);
```

delux 메서드의 사용 예시입니다. 암호화 메서드의 패러미터 타입과 반환 값 타입이 byte[]가 아닌 String입니다.

# BytesEncryptor의 구현

Spring에서 `BytesEncryptor`를 어떤 식으로 구현했는지 **`Encryptors.stronger(java.lang.CharSequence,java.lang.CharSequence)`** 메서드의 소스를 분석 후 암호화 메서드와 복호화 메서드를 단순화시켜서 보여드리겠습니다.

```java
String password = "test1234";
String salt = "01234567";

BytesKeyGenerator ivGenerator = KeyGenerators.secureRandom(16);
SecretKey secretKey = new SecretKeySpec(
        SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1")
                .generateSecret(new PBEKeySpec(password.toCharArray(), Hex.decode(salt), 1024, 256))
                .getEncoded(), "AES");

Cipher encryptor = Cipher.getInstance(AesBytesEncryptor.CipherAlgorithm.GCM.toString());
Cipher decryptor = Cipher.getInstance(AesBytesEncryptor.CipherAlgorithm.GCM.toString());

String rawText = "테스트 문자열";

byte[] iv = ivGenerator.generateKey();
encryptor.init(Cipher.ENCRYPT_MODE, secretKey, AesBytesEncryptor.CipherAlgorithm.GCM.getParameterSpec(iv));
byte[] encrypted = EncodingUtils.concatenate(iv, encryptor.doFinal(rawText.getBytes(StandardCharsets.UTF_8)));

decryptor.init(Cipher.DECRYPT_MODE, secretKey, AesBytesEncryptor.CipherAlgorithm.GCM.getParameterSpec(iv));
iv = EncodingUtils.subArray(encrypted, 0, ivGenerator.getKeyLength());
byte[] decrypted = decryptor.doFinal(EncodingUtils.subArray(encrypted, iv.length, encrypted.length));

Assert.assertEquals(rawText, new String(decrypted, StandardCharsets.UTF_8));
```

실제 소스는 `AesBytesEncryptor`, `EncodingUtils`, `CipherUtils` 등의 클래스를 활용하며, 암호화와 복호화 부분에 synchronized 블럭이 들어가지만 모두 생략하고 한 소스에서 파악할 수 있도록 했습니다. `Encryptors` 의 팩토리 메서드는 password, salt만 패러미터로 이용합니다. iv는 내부적으로 생성하고 API 사용자에게 노출하지 않습니다. iv 프로퍼티에 접근할 수 있는 메서드가 없는데 복호화는 어떻게 할까요? 암호화 메서드 호출시 위의 소스처럼 iv 값이 암호화 값 앞에 붙어서 반환되기 때문에 문제가 없습니다. iv 길이도 16바이트로 정해져있기 때문에 복호화 할 때도 암호화된 값에서 iv만큼 잘라서 iv로 이용하고 나머지 부분을 복호화합니다.

지금까지 BytesEncryptor와 TextEncryptor에 대해서 알아봤습니다. Spring 기반 프로젝트에서 AES-256을 이용한 암호화를 구현해야 한다면 직접 구현하기보단 `BytesEncryptor`또는 `TextEncryptor` 인터페이스를 써보는 건 어떨까요?

# 참조

[Spring Security Crypto Module](https://docs.spring.io/spring-security/reference/features/integrations/cryptography.html)

[org.springframework.security.crypto.encrypt.Encryptors](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/crypto/encrypt/Encryptors.html)