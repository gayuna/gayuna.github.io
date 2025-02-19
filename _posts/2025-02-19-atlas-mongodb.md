---
title: Atlas 무료 티어부터 시작하는 MongoDB 생활
categories:
  - Technologies
tags:
  - Atlas
  - MongoDB
  - DocumentDB
---

#### 자연스러웠던 Document DB의 사용 결정

땅우수학에서 학생들의 숙제를 저장할 DB를 결정할 때, MongoDB를 사용하기로 결정했습니다. 어떤 학생은 3분짜리 숙제 영상을 제출하고, 어떤 학생은 1시간짜리 영상을 제출합니다. 이 영상의 음성을 추출해 텍스트로 저장하게 되는데, 한 번 저장된 후에는 업데이트가 발생하지 않습니다. 각 숙제는 독립적이라 relation도 중요하지 않고요. 그만큼 조회도 간단한 쿼리만 일어날 것입니다. 또한, 엔트리마다 길이가 크게 차이 나는 문자열을 저장해야 하기 때문에 Document DB를 사용하는 것이 자연스러운 선택이었습니다.

#### Atlas 시작하기

그리고 이때, Atlas를 사용하기로 결정했습니다. Atlas는 MongoDB에서 직접 운영하는 서비스로, AWS, Google Cloud, Azure 등의 클라우드 환경에서 MongoDB를 손쉽게 사용할 수 있도록 지원합니다. 저 혼자 백엔드를 맡고 있는 상황에서 서버를 직접 구축하고 DB를 설정하는 것보다, 풀 패키지로 제공되는 SaaS를 활용하는 것이 더 효율적이라고 판단했습니다. 또한, Atlas에는 무료 티어(512MB)가 제공되기 때문에 PoC를 진행하며 시작하기에도 적합하다고 생각했습니다.

[Atlas 홈페이지](https://cloud.mongodb.com/)에 접속하여 회원가입한 후, 먼저 클러스터를 생성해야 합니다. 첫 번째 클러스터는 무료 티어(Free Tier)를 사용할 수 있습니다.

![Create Cluster](https://github.com/user-attachments/assets/47bb938e-75f1-45ae-a2af-4df8ec336c42)

클러스터의 이름을 적절히 지정한 후, AWS, Google Cloud, Azure 중 원하는 클라우드 프로바이더를 선택합니다. 저는 AWS를 선택했으며, 리전은 기본 추천값을 그대로 사용했습니다. (위치 기반으로 추천되는 것으로 보입니다.)

오른쪽의 Preload sample dataset 옵션을 체크한 상태로 클러스터를 생성하면, MongoDB가 제공하는 샘플 데이터를 미리 삽입한 상태로 생성되고 Task를 통해 조회 등의 연습을 해볼 수 있습니다. 저는 이 옵션을 활성화한 채로 생성한 후, 실제 서비스 운영 전에 수동으로 데이터를 삭제했습니다. 처음 MongoDB를 사용하신다면 추천합니다.

생성이 완료되면 데이터베이스 사용자 계정 및 비밀번호가 자동 생성되며, 그대로 사용하거나 수정한 후 Create 버튼을 눌러 접근 계정을 만들 수 있습니다. 이 정보는 프로젝트의 .env 파일 등에 안전하게 저장해 둡니다. (물론 나중에 계정을 새로 만들거나 비밀번호를 변경할 수도 있습니다.)

#### Atlas 연결하기

저는 개발시에는 [pymongo](https://www.mongodb.com/ko-kr/docs/languages/python/pymongo-driver/current/)를 사용하고, vscode의 [MongoDB extension](https://www.mongodb.com/products/tools/vs-code)을 사용해 데이터를 보면서 작업했습니다. 

Cluster에서 Connect를 누르면 DB와 연결할 수 있는 여러 옵션들이 나옵니다. 먼저 코드에서 연결하기 위해 Drivers를 선택합니다.

![Connect-drivers](https://github.com/user-attachments/assets/b6ba1873-b885-4d05-907b-92a6aa0342be)

원하는 언어에 맞는 드라이버 설치 방법과 샘플 코드가 제공됩니다. 특히 **View full code sample**을 토글하면, 로컬에서 실행해볼 수 있는 전체 코드가 제공되어 직접 테스트해볼 수 있습니다.

또한, 처음 Drivers 대신 MongoDB for VS Code를 선택하면, VS Code 확장 프로그램과 연결하는 방법이 안내됩니다. 이 확장은 빠른 데이터 조회가 가능하며, JavaScript를 사용해 쿼리를 테스트하고 원하는 언어로 변환할 수 있는 Playground 기능도 제공하므로, VS Code를 사용한다면 해당 확장을 활용하는 것을 추천합니다.

한편, 개발자가 아닌 분들도 데이터를 조회할 수 있는 환경이 필요했습니다. 이를 위해 새로운 인터페이스를 직접 개발하는 대신, MongoDB에서 제공하는 GUI 도구인 Compass를 활용하도록 안내했습니다. 이 때 당연히 read만 가능한 계정을 발급하는 것이 이상적이겠죠?

![Add access](https://github.com/user-attachments/assets/1ddc9e38-1833-4a08-9a54-dbadf8d04852)

Security 아래의 Database Access 항목으로 가 Add new database user를 선택합니다.

![role](https://github.com/user-attachments/assets/0455e6b2-ebec-4a2a-ab81-dedefcd27cc5)

이 때 Built-in Role 중 Only read any database를 선택함으로서 읽기만 가능한 계정을 손쉽게 만들 수 있습니다.

#### MongoDB deep dive

##### MongoDB의 계층

MongoDB의 계층은 다음과 같이 설명될 수 있습니다.

Cluster
 ├── (Shards)
 │    ├── (Replica Set)
 │    │    ├── (Instance)
 │    │    │    ├── Database
 │    │    │    │    ├── Collection
 │    │    │    │    │    ├── Document

우리가 위에서 만든 클러스터가 가장 상위 계층에 있으며, 이 안에서 데이터는 샤딩과 복제가 가능합니다. 위 표에서 Shard, Replica Set, Instance는 인프라 층위의 개념이지만, 현재 사용 중인 무료 티어에서는 직접 조정할 수 없습니다.

무료 티어에서는 M0 클러스터만 선택 가능하며, 샤딩 기능도 사용할 수 없습니다. 또한, Replica Set은 자동으로 Primary 1개, Secondary 2개로 구성되며, 이를 직접 수정할 수 없습니다. (물론 샤딩과 같은 기능은 MongoDB의 강력한 확장성 요소이므로, 대규모 확장이 필요할 때 고려해야 하지만, 이 글에서는 다루지 않겠습니다.) 따라서 무료 티어에서 애플리케이션을 개발할 경우, 가장 먼저 데이터베이스(Database)를 생성하는 과정부터 시작하게 됩니다.

여기서 데이터베이스는 하나 이상의 문서 컬렉션을 보유합니다. RDB처럼 인스턴스 내에 설치되지만, MongoDB의 데이터베이스는 서로 다른 데이터베이스 간의 레퍼런스를 지원하지 않습니다. 컬렉션(Collection)은 RDB의 테이블(Table)과 유사한 개념으로, 각 문서(Document)가 컬렉션 안에 저장됩니다. 문서(Document)는 테이블 내의 행(Row)과 비슷한 개념이며, 실제 데이터를 담고 있으며 JSON과 유사한 BSON 형식으로 표현됩니다. BSON 방식은 기계어로 변환되기 때문에 저장되기 전/읽기 전에 변환 과정을 거쳐야합니다.

##### MongoDB에서 데이터 집계하기

MongoDB의 장점이 빠른 read 속도라고 했는데, 이 때문에 단순한 CRUD는 물론 데이터 집계 및 변환을 수행하는 [Aggregation Pipeline에 대한 정보를 공식 문서](https://www.mongodb.com/ko-kr/docs/manual/core/aggregation-pipeline/)에서 제공합니다. 집계는 하나 이상의 단계(Stage)를 연결하여 수행하게 됩니다. 코드에서는 물론이고, Atlas가 제공하는 UI(웹이나 Compass)에서도 이를 시도해볼 수 있습니다. 

![pipeline](https://github.com/user-attachments/assets/ded0e66b-ab9d-47a6-8411-ef705e321ad4)

UI에서 파이프라인의 Stage를 추가하면서 각 단계별 출력 결과(Output)를 실시간으로 확인하며 파이프라인을 구성할 수 있습니다. 파이프라인이 완성되면 Run을 실행한 후 Export를 눌러 결과값을 JSON 또는 CSV 파일로 내보낼 수 있으며, 구성한 파이프라인을 저장하여 재사용할 수도 있습니다. 파이프라인을 언어별 코드 형태로 변환해주는 기능도 내재하고 있으며 특히, [Save할 때 View로 저장하면 read-only 상태로 유지되며, 항상 해당 Aggregation이 적용된 데이터를 조회](https://www.mongodb.com/ko-kr/docs/compass/current/views/)할 수 있습니다. 이를 활용하면, 사전에 read-only 계정을 부여한 비개발자 팀원이 직접 데이터를 확인할 수 있도록 구성할 수 있습니다.

##### MongoDB에서 Index 만들기

MongoDB에서도 SQL과 비슷하게 인덱스를 만들어서 검색 속도를 빠르게 할 수 있습니다. 기본적인 단일 필드, 복합 인덱스부터 [지리 공간에 대한 인덱스도 특수하게 제공](https://www.mongodb.com/ko-kr/docs/manual/core/indexes/index-types/index-geospatial/)하고, 무료 플랜을 벗어난다면 이 인덱스를 샤드 키로 사용하여 샤딩된 클러스터에서 데이터를 분할하는 [해시 샤딩](https://www.mongodb.com/ko-kr/docs/manual/core/hashed-sharding/#std-label-sharding-hashed-sharding) 기능도 제공합니다. 

다만 MongoDB는 기본적으로 빠른 읽기 속도를 위해 쓰기와 수정을 희생하고 있는데, 인덱스를 남발할 경우 비용이 더 크게 증가할 수 있으므로 자주 사용하는 쿼리에 대해서, 적절한 수준의 인덱스만 거는 것이 중요합니다.

지금까지 Atlas를 사용한다는 가정 하에 MongoDB에 대해 아주 기본적인 내용을 정리해보았습니다. MongoDB 자체에 대해 더 공부하고 싶다면 MongoDB에서 직접 운영하는 [MongoDB University](https://learn.mongodb.com/)의 강의들이 퀄리티가 좋기로 유명합니다. 여기서 공부해가면서 지식을 늘려가면 좋을 것 같습니다.
