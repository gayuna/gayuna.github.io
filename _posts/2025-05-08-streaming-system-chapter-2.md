---
title: "Streaming Systems - Chapter 2"
categories:
  - study
tags:
  - Streaming Systems
---

#### When: Early / On-Time / Late Tiggers FTW!

repeated update trigger + completeness/watermark trigger를 조합해서 사용하는 것이 이상적이다. -> Bean은  데이터가 도착하는 시점에 따라 다르게 triggering 하는 방법을 가짐 -> Early / On-Time / Late Tigger

* Early Trigger
  * 워터마크가 도달하기 전~watermark가 윈도우를 통과할 때까지 주기적으로 중간 결과를 내보냄.
  * watermark가 너무 느릴 수 있다는 단점을 보완함.
* On-Time Trigger
  * 워터마크가 해당 윈도우에 도달했을 때, 즉 정시에 트리거 됨.
  * 시스템이 판단하기에 윈도우 데이터가 대부분 도착했다고 보는 시점.
* Late Trigger
  * 워터마크 이후에 도착한 늦은 데이터에 대해 다시 트리거.

```Java
PCollection<KV<Team, Integer>> totals = input
  .apply(Window.into(FixedWindows.of(TWO_MINUTES))
               .triggering(AfterWatermark()
			     .withEarlyFirings(AlignedDelay(ONE_MINUTE))  // 1분마다 주기적으로 내보냄
			     .withLateFirings(AfterCount(1))))  // 지연된 데이터가 1개 도착할 때마다 trigger 됨
  .apply(Sum.integersPerKey());

/*
2분: window의 크기를 '이벤트 시간' 기준으로 정의 > 결과를 2분 단위로 나누어 보겠다
1분: event trigger가 발동하는 주기를 '처리 시간' 기준으로 정의 > 결과를 1분마다 중간집계 하겠다.
*/
```

[2-11 영상](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781491983867/files/assets/stsy_0211.mp4)

2-9 대비 변화:
* 두번째 window[12:02,12:04)의 'watermark too slow' 케이스에서 1분에 한번씩 주기적으로 업데이트 됨.
* 첫번째 window[12:00, 12:02)의 'heuristic watermarks too fast' 케이스에서 9 데이터가 나타났을 때 이를 반영함.
* 2-10 대비 perfect watermark와 heuristic watermark의 차이가 줄어든 것을 볼 수 있음.

heuristic watermark에서는 늦게 들어오는 데이터를 위해 일정 시간동안 window의 상태를 가지고 있어야 함. -> allowed lateness 필요.

#### When: Allowed Lateness (i.e., Garbage Collection)

이상적으로는 위의 예시처럼 이전 window에 대한 상태 정보를 모두 저장하는 것이 좋겠으나, 현실적으로 끊임 없는 데이터를 처리한다면 이를 영원히 저장하는 것은 힘듦. -> window의 수명을 정해줘야 함. : allowed lateness의 horizon을 지정.

현재시각: 12:10
watermark: 12:00 (event time이 12:00 인 것은 도착했을 것이라 가정)
allowed lateness: 5min (5분간은 더 허용해줌.)
(watermark를 기존 데이터의 표본평균, allowed lateness을 정규화 했을 때 원하는 %의 표준편차 범위 내로 하지 않을까 하는 생각)

```Java
PCollection<KV<Team, Integer>> totals = input
  .apply(Window.into(FixedWindows.of(TWO_MINUTES))
               .triggering(
                 AfterWatermark()
                   .withEarlyFirings(AlignedDelay(ONE_MINUTE))
                   .withLateFirings(AfterCount(1))
               .withAllowedLateness(ONE_MINUTE))  // 1분의 lateness도 allow
 .apply(Sum.integersPerKey());
```

[2-12 영상](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781491983867/files/assets/stsy_0212.mp4)

[12:00, 12:02)의 lateness horizon이 12:03 임을 볼 수 있음.
processing time이 12:06일 때 watermark가 12:02에 도달.
그럼 12:06에 시스템은 12:02까지의 데이터는 다 들어왔을 것이라고 가정.
그런데 allow lateness가 1분임. -> watermark가 12:03이 될 때 까지 도착한 data는 처리해주기로 함.
6 데이터는 watermark가 12:03이 되기 전에 도착함 -> 처리
9 데이터는 watermark가 12:03이 되고 나서 도착함 -> drop

#### How: Accumulation

* Discarding: 새로운 pane이 만들어 질 때마다 이전 값은 버림. downstream에서 별도로 처리를 할 때 유용.
* Accumulating: 새로운 pane이 만들어 질 때마다 이전 값을 누적.
* Acumulating and retracting: 새로운 pane이 만들어질 때마다 이전 값을 누적함과 동시에 빼는데 사용할 이전 값을 같이 보냄. "이전에 X라고 했는데 사실 Y야." downstream에서 데이터를 다시 그루핑하거나 dynamic window를 사용할 때(새로운 값이 하나 이상의 이전 window를 대신할 때) 유용함.

| Pane | Discarding | Accumulating | Accumulating & Retracting |
|------|------------|--------------|---------------------------|
| Pane 1: inputs=[3] | 3 | 3 | 3 |
| Pane 2: inputs=[8, 1] | 9 | 12 | 12, –3 |
| Value of final normal pane | 9 | 12 | 12 |
| Sum of all panes | 12 | 15 | 12 |
