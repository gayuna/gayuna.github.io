---
title: "Hello Interview - Design Youtube"
categories:
  - study
tags:
  - System Design
---

# Requirements
## Functional Requirements
* Users should be able to upload videos
* Users should be able to watch/stream videos

Scale:
* ~1M uploads/day
* 100M DAU
* Max video size of 256GB / 12 hours

## Non-Functional Requirements
* Availability >> consistency (업로드 되자마자 비디오를 볼 수 있지만 에러가 뜨기 vs 조금 기다려야 하지만 에러가 안뜨기)
* Support uploading/streaming for large videos (256GB)
* low latency streaming (<500ms) true in low bandwidth
* scalability to scale to 1M uploads/day and 100M views

# Core Entities & APIs
## Core Entities
- Video (bytes)
- Video Metadata
- User

# APIs
- Upload a video
  * POST /videos (small-sized videos)
    {
      Video,
      VideoMetadata
    }
  * POST /video (large videos) -> pre-signed URL
    {
      VideoMetadata (including size)
    }
- watch a video
  * GET /videos/{:videoId} -> Video & VideoMetadata

# High-level Design
![Image](https://github.com/user-attachments/assets/ae81ad50-e82c-4e26-8f5f-3c3f03b7f3b8)

* [한국 서비스를 기반으로 시스템 디자인 공부하기 – MY BOX](https://gayuna.github.io/system%20design/system-design-mybox/)

# Deep-dives
## support uploading / streaming for large videos
기존 디자인: S3에서 바로 download -> 1.시간이 오래 걸림 2. client에게 그만큼의 메모리 공간이 필요하고 연결도 stable 해야 함. 3. 다 다운이 되어야만 재생할 수 있음.
![Image](https://github.com/user-attachments/assets/465dfa26-8a74-411e-b6a5-574d784ee2ea)
해결법: S3에서 동영상을 쪼개서 2-10초의 chunk로 가지고 있다가 다운로드 하는 방안
이 때 streaming을 위한 chunk는 업로드에 비해 잘게 쪼갠다. (ex. 2초)

기존 디자인: 인터넷 연결이 unstable한 환경에서 접속한다면 제대로 재생이 안될 것.
![Image](https://github.com/user-attachments/assets/f4c73e3b-b9a5-4f23-948e-4f7478889356)
해결법: 여러가지 해상도/비트레이트로 미리 변환해두고 client에 맞는 파일을 내려 보내줌.
adaptive bitrate : client가 network condition을 체크. 주기적으로 체크하면서 그에 맞는 화질을 선택.

기존 디자인: hot video에 다량의 쿼리 발생 & client는 유럽에 있고 S3는 미국 서부에 있다면 latency issue 발생
![Image](https://github.com/user-attachments/assets/bcc71d94-4891-48d8-8a4d-de244c5ef78a)
해결법: DB sharding, metadata cache와 CDN 도입
