---
layout: post
title: "HTTP Security Headers"
description: "-"
date: 2021-03-18
category: WebHacking
tags: [Web, WebHacking]
comments: true
---
## Table of Contents
* [HTTP Strict Transport Security(HSTS)](#http-strict-transport-security-hsts)
* [Content-Security-Policy (CSP)](#content-security-policy-csp)
* [X-Frame-Options](#x-frame-options)
* [X-XSS-Protection](#x-xss-protection-deprecate)
  
  
  
# HTTP Strict Transport Security (HSTS)
---
간단하게 해당 WEB에 접근할 때 강제로 HTTPS로 강제적으로 접근하도록하는 헤더이다. 간혹 HTTP로 접근 시 강제로 HTTPS로 접속하도록 한다.  
HTTPS를 강제하므로써 SSL Strip 공격을 방어할 수 있는 보안 헤더, 다만 서브도메인이 허용되어 있을 경우 우회할 가능성이 존재한다.  
브라우저 내부에 HSTS List를 구성하고 있어 헤더의 정보들이 저장된다.(max-age 등)

**Values**

|이름|설명|
|------|---|
|max-age|초 단위로 설정되며, 브라우저에서 설정될 시간을 나타냄|
|includeSubdomains|해당 도메인의 서브도메인까지 설정할 것인지 나타냄|
|preload|브라우저의 Preload list에 추가하므로써 헤더가 없더라도 list에 존재할 경우 브라우저가 강제로 HTTPS로 요청도록 함|
**Example**
```
Strict-Transport-Security: max-age=<expire-time> ; includeSubDomains
Strict-Transport-Security: max-age=<expire-time>; preload;
```
<br>
<br>

# Content-Security-Policy (CSP)
---
대표적으로 XSS을 예방하는 보안 헤더로써, 태그 및 Contents의 출처에 대한 제한 및 허용 값을 헤더 값에 포함 시켜 브라우저가 응답 값의 인라인 스크립트의 실행을 제한한다.  
<meta> 태그를 이용하여 응답 값에 추가할 수 있고, 응답 헤더에 사용할 수 있다.

**지시문(일부)**

|이름|설명|
|---|---|
|default-src|디폴트를 설정|
|connect-src|ajax, websockets등 다른 URL 연결을 제한|
|script-src|script 사용 시 해당 출처를 제한|
|child-src|iframe 태그 등 inner contents를 제한|
|style-src|CSS 출처를 제한|
|font-src|Web Font의 출처를 제한|
|img-src|Image의 출처를 제한|
|media-src|media의 출처를 제한|
|object-src|object태그 사용 시 출처를 제한|

**Src Values**

|이름|설명|
|---|---|
|self|현재 도메인만 허용|
|none|모든 도메인을 제한|
|특정 도메인 및 호스트|http://www.foo.com, foo.com:80, foo.com:443|
|unsafe-inline|해당 항목은 인라인으로 사용되는 것을 허용|
|nonce-'value'|nonece-'value' 라는 속성 값을 사용할 경우, 해당 항목을 허용|

**Example (meta tag)**
```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self' trust.com *.trust.com; script-src 'nonce-aaaaaaaaaa' img-src https://*;">
```

**Example (Response Headers)**
```
Content-Security-Policy: default-src 'self' trust.com *.trust.com; script-src 'nonce-aaaaaaaaaa' img-src https://*;
```

**Example**
```html
<script src='http://foo.com/assets/js/example.js'></script> //사용불가
<script> alert('xss'); </script> //사용불가
<script nonce=aaaaaaaaaa> alert('XSS'); </script> //사용가능

<img src='http://foo.com/assets/img/foo.png'> //사용 불가
<img src='https://foo.com/assets/img/foo.png'> //사용 가능
```
<br>
<br>
  
## X-Frame-Options
---
HTML에 다른 페이지나 컨텐츠를 삽입할 수 있는 `<frame>`, `<iframe>`, `<object>` 등의 태그를 이용한 ClickJacking 공격에 대응할 수 있는 보안 헤더이다.

**Values**

|이름|설명|
|---|---|
|DENY|다른 페이지 및 컨텐츠를 표시 제한|
|SAMEORIGIN|같은 도메인의 페이지 및 컨텐츠는 허용|
|ALLOW-FROM uri|uri에 도메인을 추가하여 해당 도메인이 페이지나 컨텐츠를 허용|


**Example**
```
X-Frame-Options: deny
X-Frame-Options: sameorigin
X-Frame-Options: allow-from https://example.com/
```
<br>
<br>
  
## X-XSS-Protection (Deprecate)
---
XSS 공격을 방어하기 위한 보안 헤더로 브라우저에 내장된 XSS 필터를 통해 공격을 방지하도록한다.  
응답 값에는 정상적으로 XSS 공격하였더라도 브라우저의 해당 공격을 필터링할 경우 작동되지 않도록 한다.  이는 각각 브라우저 마다 필터나 성능의 차이로 인해 차단되는 경우는 상이하다. 그로 인해 CSP 헤더 사용을 권고한다.

**Values**

|이름|설명|
|---|---|
|0|XSS 필터링을 비활성화|
|1|XSS 필터링을 활성화, 탐지된 영역을 제거한 후 랜더링|
|1; mode=block|XSS 필터링을 활성화, 공격 탐지 시 페이지 랜더링을 중단|
|1; report=<repoting-uri>|XSS 필터링을 활성화, 공격 탐지 시 해당 uri로 리포팅 (Chromium만 지원)|

**Example**
```
X-XSS-Protection: 0; // 비활성화
X-XSS-Protection: 1; // 활성화, 랜더링
X-XSS-Protection: 1; mode=block // 활성화, 랜더링 중단
X-XSS-Protection: 1; report=foo.com // 활성화, 랜더링 중단, 해당 uri로 리포팅
```