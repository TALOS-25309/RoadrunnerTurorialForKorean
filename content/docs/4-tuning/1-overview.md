---
title: "개요"
description: "Road Runner를 사용하기 전 로봇에 맞게 시스템을 조율하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 401
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

시작하기 전에, 각 단계에서 무엇을 하는지 이해하는 것이 중요합니다.

{{<callout context="tip" title="팁" icon="outline/rocket">}}
이 페이지는 Road Runner 공식 퀵스타트의 조율 가이드를 바탕으로 작성되었으며, 
개인 경험을 바탕으로 한 추가적인 팁을 포함하고 있습니다. 
보다 자세한 내용은 [공식 가이드](https://acme-robotics.gitbook.io/road-runner/quickstart/tuning)를 
참고하세요.  
{{</callout>}}

Road Runner는 복잡한 수학적 계산을 통해 로봇의 동작을 제어하지만, 
로봇이 매끄럽게 작동하려면 반드시 **조율(Tuning) 과정**이 필요합니다. 
모터, 무게 등 로봇마다 다른 특성이 구동 방식에 영향을 미치기 때문에, 
조율 가이드를 따라 각 로봇에 맞게 구동 체계를 조정해야 합니다.

{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
로봇에 무거운 부품을 추가하는 등 큰 변화가 있을 경우, 
반드시 재튜닝이 필요합니다. 
조율 과정은 이전보다 빠르게 진행될 수 있지만, 일관된 성능을 위해 권장됩니다.  
{{</callout>}}

```kroki {type=PlantUML}

skinparam ranksep 30
skinparam dpi 125

rectangle "<b>드라이브 상수 설정</b>" as s1
rectangle "<b>데드 휠 설정</b>" as s2
rectangle "<b>로컬라이제이션 테스트</b>" as s3
rectangle "<b>직진 테스트</b>" as s5
rectangle "<b>경로 폭 조율</b>" as s6
rectangle "<b>회전 테스트</b>" as s7
rectangle "<b>로컬라이제이션 테스트</b>" as s8
rectangle "<b>팔로워 PID 조율</b>" as s9
rectangle "<b>스플라인 테스트</b>" as s10
rectangle "<b>완료</b>" as s11
rectangle "엔코더 사용" { 
rectangle "<b>주행 속도 PID 조율</b>" as s4_y
}
rectangle "엔코더 미사용" { 
rectangle "<b>주행 피드포워드 조율</b>" as s4_n
}

(s1) --> (s2)
(s2) --> (s3)
(s3) --> (s4_y)
(s3) --> (s4_n)
(s4_y) --> (s5)
(s4_n) --> (s5)
(s5) --> (s6)
(s6) --> (s7)
(s7) --> (s8)
(s8) --> (s9)
(s9) --> (s10)
(s10) --> (s11)
```

**각 단계를 완료한 후에 다음 단계로 진행하세요.**

---

## 피드포워드(Feedforward)란 무엇인가?

피드 포워드 속도 제어는 전압을 속도로 변환하는 함수를 생성하는 개방 루프 시스템입니다. 
이 과정에서 구동 중 실시간 피드백이 부족하다는 문제가 있지만, 
병진 이동 PID가 이를 보완하여 에러를 수정합니다.

---

## 주행 속도 PID (Drive Velocity PID)는 어디에 있나요?

주행 속도 PID는 더 이상 사용되지 않습니다. 
그 이유는 [여기](/drive-velocity-pid-tuning.html#why-is-drivevelocitypid-not-used)에서 확인할 수 있습니다.  
드라이브 인코더 또는 오도메트리 사용 여부와 관계없이 피드포워드 방식을 사용하세요.

---

## 드라이브 상수 설정

드라이브 상수(Drive Constants) 파일에는 로봇의 물리적 특성과 관련된 모든 정보가 포함됩니다. 
예를 들어, 모터의 최대 RPM, 바퀴 반지름 등이 포함됩니다.
이 단계에서 발생하는 가장 심각한 오류는 보통 드라이브 상수에서 시작됩니다.  
예를 들어, 로봇이 지정된 거리의 절반만 이동한다면, 이는 드라이브 상수의 문제일 가능성이 높습니다.

드라이브 상수 사용에 대한 자세한 내용은 [드라이브 상수 설정 페이지](../드라이브-상수-설정-drive-constant/)에서 확인할 수 있습니다.

---

## 데드 휠 설정

로봇에 데드 휠이 있다면, 드라이브 상수를 편집한 후 데드 휠을 설정해야 합니다.
데드 휠이 없다면 이 단계를 무시하세요.  
데드 휠에 대해 잘 모르겠다면 [FAQ](../../1-about-roadrunner/자주-묻는-질문-faq/#2-데드-휠오도메트리란-무엇인가요)의 예제를 참고하세요.

데드 휠 조율은 로컬라이제이션 테스트에서 수행됩니다. 데드 휠의 정확한 조율은 
로봇의 정확한 로컬라이제이션 및 경로 추적을 위해 매우 중요합니다.  
로봇이 2개의 데드 휠을 사용하는지, 3개의 데드 휠을 사용하는지에 따라 설정이 달라집니다. 
차이를 모르겠다면 [FAQ](../../1-about-roadrunner/자주-묻는-질문-faq/#3-두-바퀴와-세-바퀴-오도메트리의-차이점은-무엇인가요)를 참고하세요.

자세한 내용은 [데드 휠 설정 페이지](../데드-휠-설정-dead-wheels)에서 확인할 수 있습니다.

---

## 로컬라이제이션 테스트 1

{{< callout context="caution" title="주의" icon="outline/alert-triangle" >}}
이 첫 번째 로컬라이제이션 테스트는 데드 휠 로컬라이제이션을 
테스트 및 조율하기 위한 것입니다. 데드 휠를 사용하지 않기로 결정했다면 이 단계를 건너뛰세요.
{{< /callout >}}

로컬라이제이션 테스트를 실행하고 로봇을 필드에서 이동시켜 로컬라이제이션과 관련된 문제를 찾으세요.
데드 휠 로컬라이제이션은 데드 휠을 설정한 후 조율해야 합니다.  
로컬라이제이션의 정확성은 경로 추적의 정확성에 큰 영향을 미칩니다.

데드 휠을 사용하지 않는 경우, 이후 단계에서 로컬라이제이션 테스트를 수행합니다.

자세한 내용은 [데드 휠 조절 페이지](../데드-휠-설정-dead-wheels)에서 확인할 수 있습니다.

---

## 주행 피드포워드 조율

피드포워드 방식을 선택했다면 피드포워드 상수를 조율해야 합니다.  
공식 Quickstart는 자동 튜너와 수동 튜너를 제공합니다. 하지만 일부 사용자는 자동 튜너가 최적의 결과를 제공하지 않는다고 느낄 수 있습니다.

**자동 조율 과정**은 다음과 같습니다(공식 문서에서 발췌):

> `kV`와 `kStatic`을 찾기 위해 로봇은 전압을 천천히 증가시키는 정적 램프 테스트를 실행합니다. 이 과정에서 속도와 전압이 기록됩니다. 속도 대 전압 그래프에서 `kV`는 기울기, `kStatic`은 Y절편입니다.  
> 다음으로 `kA`를 찾기 위해 로봇은 정지 상태에서 급가속을 시도합니다. 이 과정에서 가속도, 속도, 전압이 기록됩니다. 가속도 대 전압 그래프에서 기울기는 `kA`입니다.

자동 튜너는 `kA` 값을 매우 낮게 산출하는 경향이 있습니다. 따라서 수동 조율 방식을 추천합니다.

자세한 내용은 [주행 피드포워드 페이지](../주행-피드포워드-조율-driving-feedforward-tuner)에서 확인할 수 있습니다.

---

## 주행 속도 PID 조율 [사용하지 않음!]

{{< callout context="danger" title="경고" icon="outline/alert-octagon" >}}
이 조율 방법을 따르지 마십시오. 피드 포워드 조율을 대신 사용하세요.
{{< /callout >}}

`DriveVelocityPIDTuner`는 Rev Hub의 내장 모터 속도 제어기(`RUN_USING_ENCODER` 모드)를 조율하는 데 사용됩니다. 
PIDF 계수를 최적화하여 일관된 동작을 보장하는 것이 중요합니다.  
PIDF 계수는 로봇의 무게에 영향을 미치는 주요 변경 사항 이후에 반드시 조정해야 합니다.

주행 속도 PID 조율 과정은 [주행 속도 PID Tuning 페이지](../주행-속도-pid-조율-driving-velocity-pid-tuner)에 자세히 설명되어 있습니다. 
PIDF 값을 조정하여 원하는 동작을 얻을 수 있습니다.  
공식 Road Runner 문서에서는 "약간의 진동이 발생하더라도 위상 지연을 제거하는 것을 우선시하라"고 권장하지만, 
개인적으로는 특히 정지 상태에서 진동을 최소화하는 것이 더 낫다고 생각합니다.  
위상 지연을 제거하면 고속에서 매우 불안정한 동작을 유발할 수 있습니다. 
이는 Rev Hub의 모터 제어 방식 때문일 가능성이 높습니다.  
더 자세한 기술적 내용을 알고 싶다면 [FTC Discord](https://discord.gg/first-tech-challenge)를 통해 문의하세요.  
제 개인적인 조언은 진동을 최소화하고 병진 이동 PID를 통해 위상 지연 문제를 해결하는 것입니다.

---

## 직진 테스트

직진 테스트는 피드포워드/속도 PID 조율의 효과를 확인하는 데 사용됩니다. 
`StraightTest` OpMode를 여러 번 실행하여 로봇이 일정한 거리 내에서 안정적으로 이동하는지 확인하세요.  
로봇이 매번 정확히 같은 위치에 도달하지 않아도 됩니다. 
이후 로컬라이제이션을 통해 폐쇄 루프 피드백이 활성화될 것입니다.

데드 휠 없이 엔코더만 사용하는 경우, 이 단계에서 엔코더 로컬라이제이션을 튜닝하게 됩니다.

자세한 내용은 [직진 테스트 페이지](../직진-테스트-straight-test)에서 확인할 수 있습니다.

---

## 경로 폭 조율

경로 폭(Track Width)는 한 바퀴에서 평행한 다른 바퀴까지의 거리입니다. 이는 물리적으로 측정할 수 있지만, 실제 경로 폭는 여러 요인으로 인해 물리적 측정값과 다를 수 있습니다.  
이를 보정하기 위해 `TrackWidthTuner` OpMode를 실행하여 경험적 경로 폭를 계산합니다.

`TrackWidthTuner`는 원하는 경험적 경로 폭에 1인치 정도의 오차 범위 내에서 도달합니다. `TurnTest` OpMode를 실행하고 드라이브 상수에서 경로 폭를 수정하여 수동으로 조정해야 할 수도 있습니다.

자세한 내용은 [Track Width Tuning 페이지](../경로-폭-조율-track-width-tuner)에서 확인할 수 있습니다.

---

## 회전 테스트

회전 테스트를 실행하여 경로 폭가 올바른지 확인하세요.

자세한 내용은 [회전 테스트 조율 페이지](../회전-테스트-조율-turn-test-tuning)에서 확인할 수 있습니다.

---

## 로컬라이제이션 테스트 2

{{< callout context="caution" title="주의" icon="outline/alert-triangle" >}}
두 번째 로컬라이제이션 테스트는 엔코더를 사용하는 로컬라이제이션을 테스트하기 위한 것입니다. 엔코더를 사용하지 않기로 결정했다면 이 단계를 건너뛰세요.
{{< /callout >}}

로컬라이제이션 테스트를 실행하고 로봇을 필드에서 이동시켜 엔코더를 사용하는 로컬라이제이션과 관련된 문제를 찾으세요.  
엔코더를 사용하는 로컬라이제이션은 이전 단계에서 조율되어야 합니다. 로컬라이제이션의 정확성은 경로 추적의 정확성에 큰 영향을 미칩니다.

두 번째 로컬라이제이션 테스트는 엔코더를 사용하는 구동의 정확성을 테스트하는 데 사용됩니다.  
데드 휠을 사용하여 로컬라이제이션을 조율했다면 이 단계를 건너뛰세요.

자세한 내용은 [로컬라이제이션 테스트 페이지](../로컬라이제이션-테스트-localization-test)에서 확인할 수 있습니다.

---

## 팔로워 PID 조율

이 단계에서는 회전 PID와 병진 이동 (x/y) PID 두 가지를 조율합니다.  
이 PID는 폐쇄 루프 피드백 제어를 활성화하여 정확한 경로 추적을 보장합니다.  
`BackAndForth` OpMode를 실행하여 회전 및 병진 이동 PID를 대략적으로 조율한 후, `FollowerPIDTuner` OpMode를 사용해 세부적으로 조정합니다.

자세한 내용은 [팔로워 PID 조율 페이지](../팔로워-pid-조율-follower-pid-tuner)에서 확인할 수 있습니다.

---

## 스플라인 테스트

모든 조율이 완료된 후, 로봇은 스플라인 경로를 정확하게 따라야 합니다.  
스플라인 테스트가 실패하면 문제를 식별하고 해당 단계로 돌아가 다시 조율하세요.  
어려움을 겪고 있다면 [FTC Discord](https://discord.gg/first-tech-challenge)에서 도움을 요청하세요!

자세한 내용은 [스플라인 테스트 페이지](../스플라인-테스트-spline-test)에서 확인할 수 있습니다.