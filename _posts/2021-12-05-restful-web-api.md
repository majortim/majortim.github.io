---
layout: post
title:  "RESTful Web API 도서 내용 정리"
description: "RESTful Web API - 웹 API를 위한 모범 전략 가이드(원제 RESTful Web APIs) 정리"
date:   2021-12-05 12:00:00 +0900
categories: Web
excerpt_separator: <!--more-->
---

RESTful Web API - 웹 API를 위한 모범 전략 가이드(원제 RESTful Web APIs: Services for a Changing World)의 내용을 정리해봤습니다.

<!--more-->

# 표준의 종류

명목 표준 < 개인 표준 / 기업 표준 < 개방형 표준 < RFC

- 명목 표준

명목 표준은 행동 양식이다. 구글 API, 페이스북 API 등이 해당한다. 문서화는 할 수 있지만 다른 사람도 동일한 방식으로 해야한다는 표준의 핵심 가정이 빠져있다. 이 설계에 맞춘 작업을 하려면 이 설계를 이해해야 한다. 명목 표준을 재사용하는 것은 API가 표준을 따른다고 하지 않고 복제본이라고 부른다.

# HATEOAS

**Hypermedia as the Engine of Application State**란 하이퍼미디어 제약 조건이다.  용어에서 말하는 애플리케이션 상태란 클라이언트가 현재 있는 페이지를 말한다. 하이퍼미디어는 HTML 링크와 폼 같은 것들을 부르는 일반 용어다. 그래서 전체 용어에 대한 풀이는 클라이언트가 현재 어디에 있고 어디로 연결될 수 있는지를 말한다.

# 안전하지 않은 HTTP 메서드

안전하지 않은 HTTP 메서드는 해당 요청을 보냈을 때와 보내지 않았을 때의 결과(서버에 일어나는 상태 변화)가 다르다. 안전하지 않은 메서드로는 POST, PATCH 등이 있다.

# 멱등성(Idempotent)

동일한 조건 아래에서는 여러 번 시도한다고 처음 시도와 나중 시도의 결과가 달라지지 않는다.

# 리소스(Resource)

웹 페이지, 사람, 이름 등 무엇이든 리소스가 될 수 있지만 그 리소스는 자신의 URI를 가져야 한다.

# 표현(Representation)

리소스의 상태를 나타낸다. HTTP 요청이나 응답의 엔티티 바디로 볼 수도 있고 요청이나 응답 메시지 전체를 표현이라고 볼 수도 있다. 서버가 표현을 클라이언트에 보낼 때는 서버의 상태를 나타내는 것이고, 클라이언트가 서버에 표현을 보낼 때는 서버의 상태를 수정하려는 것이다.

# 자기 서술적 메시지(Self-descriptive Messages)

요청이나 응답 메시지를 이해하기 위해 필요한 모든 정보가 그 메시지 자체에 포함되어 있거나 적어도 그 링크가 있어야 한다.

# 무상태(Statelessness)

클라이언트가 요청을 보내는 그 순간을 제외하고는 서버는 클라이언트의 존재를 모른다.

# 프로필(Profile)

미디어 유형을 아는 것만으로는 알 수 없는 문서의 의미 체계를 설명한다.

# 의미 체계 서술자(Semantic Descriptor)

리소스 상태의 각 조각을 명명하는 짧은 문자열. 프로파일로 보통 사람이 읽을 수 있는 설명으로 주어진다. **RESTful Web API** 책을 위해 만들어진 단어로 표준 용어는 아니다.

# API 설계 절차

1. API를 통해 제공할 정보를 열거한다. 이 정보들이 의미 체계 서술자가 된다. 이 정보들은 필요에 따라 계층화할 수도 있다. 현실 세계의 개체를 참조하고 더 자세한 동작을 정의할 수 있다.

2. API 상태 다이어그램을 그린다. 화살표를 이용해 HTTP 요청이 작동시키는 상태 전이를 표현한다. 상태 전이에 HTTP 메서드를 할당할 필요는 없지만 상태 전이가 안전한지, 멱등한지 여부는 계속 알고있어야 한다(원서에서는 you should keep track of 라고 했고, 번역서에서는 주시해야 한다라고 써져있다). 의미 체계 서술자와 그 연결 관계가 만족스러울 때까지 1단계와 2단계를 반복한다.

3. 기존에 있던 프로필의 문자열을 의미에 맞게 조정한다. 조정하면서 이전 단계들을 반복할 수 있다.

4. 프로토콜 의미 체계, 애플리케이션 의미 체계와 호환되는 미디어 유형을 선택한다.

5. 애플리케이션 의미 체계를 서술하는 프로필을 작성한다. ALPS 문서, JSON-LD 콘텍스트 등을 이용할 수 있다.

6. 코드를 구현한다.

7. API를 게시한다.

# 참조

**[RESTful Web API](https://blog.insightbook.co.kr/2015/09/04/%ec%9b%b9-api-%ec%96%b4%eb%96%bb%ea%b2%8c-%eb%a7%8c%eb%93%a4%eb%a9%b4-%ec%a2%8b%ec%9d%84%ea%b9%8c-%e3%80%8erestful-web-api%e3%80%8f/)**

[RFC 6906 / The 'profile' Link Relation Type, MARCH 2013](https://www.rfc-editor.org/info/rfc6906)

[Link relation](https://en.wikipedia.org/wiki/Link_relation)