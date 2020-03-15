---
layout: page
title: Projects
permalink: /projects/
---

#### **내친구서울 홈페이지**
![portal](/assets/img/mfs.png)

기간: 2019.06. ~ 2019.12.\\
개발 환경/사용 기술: Java, eGovFrame, Spring Framework, JSP, MyBatis, HTML5, CSS, JavaScript, jQuery, Oracle Database, Wildfy, Apache HTTP Server

서울시의 어린이신문 '내친구서울' 홈페이지를 구축 및 운영했습니다.

#### **대학 종합정보시스템 학사/행정/연구 프로그램**

![portal](/assets/img/portal.png)

기간: 2016.07. ~ 2018.05.\\
개발 환경/사용 기술: ActionScript3, FLEX, Java, eGovFramework, Oracle Database, Tomcat

한국기술교육대학교 온라인 종합정보시스템상에서 쓰는 행정 프로그램을 개발 및 운영했습니다. 사용한 언어는 액션스크립트와 자바, 개발 도구는 이클립스와 FLEX SDK, 자체 프레임워크, 전자정부 표준프레임워크 등입니다. 인사, 구매, 급여, 강의, 장학금 등 다양한 종류의 프로그램 개발에 참여했으며, 참여한 프로그램 중 대표적인 것으로 시내 출장 등록관리, 외부강의 신청관리, 기술연구원 근무성적 평가, 근로 장학 신청관리 등이 있습니다. 

<!-- --- -->

#### **UMS**

![portal](/assets/img/ums.png)

기간: 2017.07. ~ 2017.09.\\
개발 환경/사용 기술: Java, Spring Framework, Thymeleaf, MyBatis, HTML5, CSS, JavaScript, Bootstrap, jQuery, Oracle Database, Tomcat

교내 통합 메시징 시스템(Unified Messaging System, UMS) 고도화에 참여했습니다. 기존에 있던 시스템상에선 휴대전화 문자메시지만 보낼 수 있는 서비스를 제공했는데, 기존 시스템에서 모바일 환경에 맞게 UI를 변경하는 동시에 카카오톡 알림톡 기능과 설문 시스템, 메시지 템플릿 기능이 추가됐습니다. 제가 맡은 부분은 메시지를 받을 구성원들을 조건에 따라 조회하는 맞춤공지라는 메뉴입니다. 학부생 검색, 교직원 검색을 포함한 나머지 다섯 개의 페이지를 구현했습니다.

#### **KOBUS**

![portal](/assets/img/kobus.png)

기간: 2015.03. ~ 2015.09.\\
개발 환경/사용 기술: Java, Android, Google Maps API

셔틀버스의 위치정보를 실시간으로 조회할 수 있는 Android 애플리케이션입니다. 예전에 통학할 때 셔틀버스가 정류장에 도착하는 시간이 일정하지 않아서 불편했는데, 버스가 지나갔는지 확인할 수 있는 앱이 생기면 편리할 거 같아서 졸업작품 주제로 선택했습니다. Android 디바이스의 GPS, NFC(Near Field Communication, 근거리 무선 통신) 센서 기술을 이용했습니다. 셔틀버스 위치추적 및 승차장 정보 조회, 시간표 조회, 도착 전 알림, 버스 내부 승객 수 조회 등의 기능을 제공합니다. 저는 서버-클라이언트 통신, 버스 내부 승객 수 조회 부분을 맡았습니다. 서버는 Java를 이용해 만들었고 TCP Socket 방식이며 클라이언트 접속 하나당 하나의 스레드를 만들어 사용하는 멀티스레드 방식입니다. 버스 내부 승객 수 조회는 Android의 NFC 기능을 이용해 학생증을 단말기(휴대전화)에 갖다 대면 카운트하는 방식입니다. 이 부분은 Github에 있는 오픈 소스를 이용해 만들었습니다.
