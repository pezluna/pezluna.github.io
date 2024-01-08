---
layout: post
title:  "pyPlc Analysis 1"
summary: "How to launch pyPlc on Win10"
author: sjhan
date: '2024-01-08 00:00:00'
category: EVC
thumbnail: /assets/img/posts/code.jpg
keywords: EVC, EVSE
permalink: /blog/pyPlc-Analysis_1/
usemathjax: true
---

# pyPlc 분석
윈도우 10에서 EVC 에뮬레이터인 pyPlc를 시뮬레이션 모드로 실행하고자 하였으나, 실행이 되지 않는 문제가 있었다. 이를 해결하기 위해 pyPlc의 소스코드를 분석하였다.

## pyPlc 실행
pyPlc를 윈도우 10 환경에서 실행하기 위해서는 다음과 같은 과정을 거쳐야 한다.
1. 필요 도구 설치(Python, Pcap-ct 등)
2. pyPlc 소스코드 다운로드
3. OpenV2G 빌드
4. pyPlc 실행

상기 과정은 [pyPlc 설치 가이드](https://github.com/uhi22/pyPLC/blob/4f96593177a138994036c2742d455b6481d5f5a4/doc/installation_on_windows.md)에 자세히 설명되어 있다.

그러나 이 과정을 거쳐도 pyPlc가 실행되지 않는 문제가 있었다.

(한글 문자에 대해 `ord()`를 사용하여 발생하는 문제는 수정하기 쉽기에 생략한다.)

![Error 01](/assets/img/posts/pyplc/error_01.png)

## 에러 메시지 분석
에러 메시지를 보면 `hardwareInterface.py`의 `initPorts()` 함수에서 적절하지 않은 인자를 넘겨주어서 발생하는 문제임을 유추할 수 있다.

해당 함수는 다음과 같다.
```python
self.canbus = can.interface.Bus(bustype='socketcan', channel='can0', can_filters=filters)
```

여기서 사용된 socketcan은 리눅스에서 사용되는 소켓 기반의 CAN 통신 프로토콜로, 기본적으로 윈도우에서는 socketcan을 지원하지 않는다.

그런데 pyPlc는 윈도우10에서 실행이 가능하다고 doc에 명시되어 있다.

![Doc 01](/assets/img/posts/pyplc/doc_01.png)

