---
layout: post
title:  "pyPLC Analysis 1"
summary: "How to launch pyPLC on Win10"
author: sjhan
date: '2024-01-08 15:18:00'
category: EVC
thumbnail: /assets/img/posts/pyplc/pyPLC_thumbnail_01.png
keywords: EVC, EVSE
permalink: /blog/pyPLC-Analysis_1/
usemathjax: true
---

# pyPLC 분석
윈도우 10에서 EVC 에뮬레이터인 pyPLC를 시뮬레이션 모드로 실행하고자 하였으나, 실행이 되지 않는 문제가 있었다. 이를 해결하기 위해 pyPLC의 소스코드를 분석하였다.

## pyPLC 실행
pyPLC를 윈도우 10 환경에서 실행하기 위해서는 다음과 같은 과정을 거쳐야 한다.
1. 필요 도구 설치(Python, Pcap-ct 등)
2. pyPLC 소스코드 다운로드
3. OpenV2G 빌드
4. pyPLC 실행

상기 과정은 [pyPLC 설치 가이드](https://github.com/uhi22/pyPLC/blob/4f96593177a138994036c2742d455b6481d5f5a4/doc/installation_on_windows.md)에 자세히 설명되어 있다.

그러나 이 과정을 거쳐도 pyPLC가 실행되지 않는 문제가 있었다.

(한글 문자에 대해 `ord()`를 사용하여 발생하는 문제는 수정하기 쉽기에 생략한다.)

![Error 01](/assets/img/posts/pyplc/error_01.png){: width=90% height=90% .center-image}

## 에러 메시지 분석
에러 메시지를 보면 `hardwareInterface.py`의 `initPorts()` 함수에서 적절하지 않은 인자를 넘겨주어서 발생하는 문제임을 유추할 수 있다.

해당 함수는 다음과 같다.
```python
self.canbus = can.interface.Bus(bustype='socketcan', channel='can0', can_filters=filters)
```

여기서 사용된 socketcan은 리눅스에서 사용되는 소켓 기반의 CAN 통신 프로토콜로, 기본적으로 윈도우에서는 socketcan을 지원하지 않는다.

그런데 pyPLC는 윈도우 10에서 실행이 가능하다고 doc에 명시되어 있다.

![Doc 01](/assets/img/posts/pyplc/doc_01.png){: width="90%" height="90%" .center-image}

그래서 socketcan을 윈도우 10에서 사용할 수 있는 방법을 찾아보았다.

가장 먼저 찾은 방법은 한 [레딧의 글](https://www.reddit.com/r/CarHacking/comments/ot3gjf/socketcancanutils_on_windows/)이었다.

wls2를 사용하여 윈도우 10에서 리눅스 환경을 구축하고, socketcan을 사용하는 방법이다.

그러나 이 방법은 윈도우 10에서 온전히 pyPLC를 실행하는 방법이 아니라고 생각했기에, 다른 방법을 찾아보았다.

## socketcan on Windows
결론적으로, 나는 임시방편으로 pyPlc.ini 파일을 수정하여 pyPLC를 실행하였다.

오류가 발생한 부분을 보면, 다음과 같은 조건문이 있음을 알 수 있다.
```python
def initPorts(self):
    if (getConfigValue("charge_parameter_backend") == "chademo"):
        filters = [
            {"can_id": 0x100, "can_mask": 0x7FF, "extended": False},
            {"can_id": 0x101, "can_mask": 0x7FF, "extended": False},
            {"can_id": 0x102, "can_mask": 0x7FF, "extended": False}]
        self.canbus = can.interface.Bus(bustype='socketcan', channel='can0', can_filters=filters)

    # ...
```

위 조건문을 보면, `charge_parameter_backend`가 `chademo`일 때에만 `canbus`를 초기화하고 있다.

해당 변수는 `pyPlc.ini` 파일에 정의되어 있으며, `charge_parameter_backend`에 대한 설명은 다음과 같다.
```ini
# Set backend for obtaining charging parameters, we start with CHAdeMO CAN for now
# Need to make a simulator device and maybe a celeron device?
# Possible values:
#  chademo: pyPLC is used as bridge between a CCS charger and a CHAdeMO* car.
#           Limitations/explanations here: https://openinverter.org/forum/viewtopic.php?p=57894#p57894 and
#           https://openinverter.org/forum/viewtopic.php?t=1063 (Is it possible to make a CCS to CHAdeMO adapter?)
#  none: all other use cases
charge_parameter_backend = chademo
```

`pyPlc.ini` 파일의 설명에 따르면 기본적으로 pyPLC는 CHAdeMO 표준을 사용하는 EV와 EVC를 연결하는 브릿지 역할을 수행하며, 이를 위해 socketcan을 사용한다.

그러나 현재로서는 시뮬레이션 환경에서 pyPLC를 실행하고자 하기 때문에, `charge_parameter_backend`를 `none`으로 설정하고 pyPLC를 실행하면 정상적으로 실행된다.

![Success 01](/assets/img/posts/pyplc/success_01.png){: width="80%" height="80%" .center-image}

## 결론
pyPLC는 기본적으로 CHAdeMO 표준을 사용하는 EVSE와 EV를 연결하는 브릿지 역할을 수행하며, 이를 위해 socketcan을 사용한다.

이러한 pyPLC를 시뮬레이션 환경에서 실행하고자 할 때에는 굳이 CHAdeMO 표준을 사용하지 않아도 되기 때문에, `pyPlc.ini` 파일의 `charge_parameter_backend`를 `none`으로 설정하면 정상적으로 실행된다.