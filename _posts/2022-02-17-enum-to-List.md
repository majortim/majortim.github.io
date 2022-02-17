---
layout: post
title:  "enum의 필드를 Map타입 List로 변환"
description: "enum의 필드를 Map타입 List로 변환하는 제네릭 메서드를 정의하고 사용합니다."
date:   2022-02-17 23:45:00 +0900
categories: Java
excerpt_separator: <!--more-->
---

enum의 필드를 Map타입 List로 변환하는 방법을 소개합니다. 제네릭 메서드와 함수형 인터페이스를 이용합니다.

<!--more-->


```java
package io.hurem.model;

public interface EnumModel {
    String getKey();
    String getValue();
}
```

enum에서 구현할 인터페이스. getKey()는 enum 필드의 이름을, getValue()는 단일 프로퍼티를 가져온다.

```java
package io.hurem.domain.employees;

import io.hurem.model.EnumModel;

import java.util.Collections;
import java.util.Map;
import java.util.Optional;
import java.util.function.Function;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public enum EmployeeStatusType implements EnumModel {
    PRESENT("재직"),
    ABSENT("일반휴직"),
    PARENTAL("육아휴직"),
    UNPAID("무급휴직"),
    SUSPENDED("정직"),
    RETIRED("퇴직"),
    UNKNOWN("알 수 없음");

    private final String status;

    EmployeeStatusType(String status) {
        this.status = status;
    }

    @Override
    public String getKey() {
        return name();
    }

    @Override
    public String getValue() {
        return status;
    }

    private static final Map<String, EmployeeStatusType> types = Collections.unmodifiableMap(Stream.of(values()).collect(Collectors.toMap(EmployeeStatusType::getValue, Function.identity())));

    public static EmployeeStatusType find(String description) { return Optional.ofNullable(types.get(description)).orElse(UNKNOWN); }

}
```

EnumModel을 구현한 enum. 프로퍼티 값에 해당하는 enum을 찾는 메서드 find를 Map타입을 이용해 구현했다.

```java
package io.hurem.util;

import io.hurem.model.EnumModel;
import org.springframework.stereotype.Component;

import java.util.*;
import java.util.function.Function;
import java.util.stream.Collectors;

@Component
public class EnumUtils {
    @SafeVarargs
    public final <E extends Enum<E> & EnumModel> List<Map<String, String>> getEnumList(final Class<E> enumClass, E... exclusions) {
        List<E> list =  new ArrayList<>(Arrays.asList(enumClass.getEnumConstants()));
        Optional.of(exclusions).ifPresent(arr -> list.removeAll(Arrays.asList(arr)));

        Function<E, Map<String, String>> enumModelToMap = (model) -> {
            Map<String, String> result = new HashMap<>();

            result.put("name", model.getKey());
            result.put("value", model.getValue());
            return result;
        };

        return list.stream().map(enumModelToMap).collect(Collectors.toList());
    }
}
```

핵심 클래스. 메모리 문제를 생각해서 메서드를 static으로 정의하지 않는 대신 EnumUtils에 @Component를 붙여 빈으로 등록하도록 했다.

특정 enum에 종속되지 않도록 제네릭 메서드로 구현했다. 타입 패러미터 부분에 들어간 &에 주목해야 하는데, E 타입은 Enum이면서 동시에 EnumModel 인터페이스를 구현함을 의미한다.

메서드의 첫 번째 인자는 사용하려는 enum의 클래스 타입이다. 첫 번째 인자의 getEnumConstants() 메서드로 Enum의 배열을 가져오며 Arrays.asList() 메서드를 이용해 List로 변경한다. getEnumConstants() 메서드 호출이 반환하는 배열은 원소 순서가 enum에 선언된 필드 순서와 같다.

메서드의 두 번째 인자는 enum에서 가져오지 않을 필드의 가변인자이다. 두 번째 인자가 null이 아니면 List에서 해당 필드들을 제거한다.

이제 Stream과 Function 인터페이스를 이용해 List의 원소들을 HashMap으로 매핑하고 다시 List로 취합해서 메서드 반환 값으로 전환한다.

```java
package io.hurem.domain.util;

import io.hurem.domain.employees.EmployeeStatusType;
import io.hurem.util.EnumUtils;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@Slf4j
@SpringBootTest
public class EnumUtilsTest {
    @Autowired
    EnumUtils enumUtils;
    @Test
    public void listTest() {
        log.debug("list1: {}", enumUtils.getEnumList(EmployeeStatusType.class));
        log.debug("list2: {}", enumUtils.getEnumList(EmployeeStatusType.class, EmployeeStatusType.ABSENT, EmployeeStatusType.UNKNOWN));
    }
}
```

EnumUtils가 생각한대로 동작하는지 테스트하는 코드다.

```java
2022-02-17 23:26:15,561 DEBUG io.hurem.domain.util.EnumUtilsTest : list1: [{name=PRESENT, value=재직}, {name=ABSENT, value=일반휴직}, {name=PARENTAL, value=육아휴직}, {name=UNPAID, value=무급휴직}, {name=SUSPENDED, value=정직}, {name=RETIRED, value=퇴직}, {name=UNKNOWN, value=알 수 없음}]
2022-02-17 23:26:15,563 DEBUG io.hurem.domain.util.EnumUtilsTest : list2: [{name=PRESENT, value=재직}, {name=PARENTAL, value=육아휴직}, {name=UNPAID, value=무급휴직}, {name=SUSPENDED, value=정직}, {name=RETIRED, value=퇴직}]
```

의도한대로 List를 반환하며 제외 항목도 잘 처리한다. 순서도 enum 클래스에 정의한 순서대로 나온다.

# 참조

[Enum 활용 & Enum 리스트 가져오기](https://jojoldu.tistory.com/122)

[Enum 조회 성능 높여보기](https://pjh3749.tistory.com/279)