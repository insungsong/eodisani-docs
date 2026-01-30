# '어디사니(Eodisani)' - 1인 풀스택 개발 포트폴리오 대본 (Technical Deep Dive Ver.)

## Slide 1: 표지 (Intro)
**[화면 구성]**
- 배경: IDE 코드 화면(복잡한 SQL 쿼리)과 어디사니 앱 지도 화면이 오버랩된 이미지
- 제목: 1인 개발로 완성한 하이퍼로컬 커뮤니티, '어디사니'
- 부제: PostGIS 기반 공간 분석 엔진부터 MVT 타일 서빙까지
- 발표자: 송인성 (Backend Engineer & Full Stack Developer)

**[대본]**
안녕하십니까. 기술적 깊이로 비즈니스 문제를 해결하는 백엔드 엔지니어 **송인성**입니다.
오늘 소개할 '어디사니'는 단순한 위치 기반 SNS가 아닙니다.
기획부터 인프라까지 1인 개발로 진행하며, **PostGIS를 활용한 실시간 MVT(Vector Tile) 생성**, **비동기 논블로킹 로그 시스템**, 그리고 **대용량 공간 데이터 최적화**까지 직접 구현한 하이퍼로컬 플랫폼입니다.
현업 수준의 아키텍처 고민과 기술적 해결 과정을 중점으로 말씀드리겠습니다.

---

## Slide 2: 개발 목표 - "지도 위에 10만 개의 데이터를 어떻게 뿌릴까?" (Goal)
**[화면 구성]**
- 좌측: 수많은 마커로 뒤덮인 느린 지도 앱 (Bad Case)
- 우측: 깔끔하게 클러스터링되고 빠르게 로딩되는 어디사니 앱 (Good Case)
- 키워드: "High Performance", "Spatial Intelligence", "Scalability"

**[대본]**
프로젝트의 시작은 명확한 기술적 도전이었습니다.
"서울시 내의 모든 건물과 커뮤니티 정보를 모바일 지도에 표현한다면?"
수만 개의 데이터를 단순 JSON API로 주고받는 것은 불가능했습니다.
저는 이 문제를 해결하기 위해 백엔드에서 직접 **Vector Tile**을 생성하여 서빙하고, 프론트엔드에서는 GPU 가속을 활용해 렌더링하는 고성능 지도 엔진을 구축하는 것을 목표로 삼았습니다.

---

## Slide 3: 전체 기술 스택 (Tech Stack)
**[화면 구성]**
- **Core**: NestJS, Node.js, TypeScript
- **Database**: PostgreSQL + **PostGIS** (Spatial Indexing)
- **Map Engine**: Mapbox Vector Tile (MVT) utilizing `ST_AsMVT`
- **Frontend**: React Native, Expo
- **Infra**: AWS (RDS, EC2), Redis (Tile Caching)

**[대본]**
기술 스택은 단순히 '할 줄 아는 것'이 아닌 '문제 해결에 최적화된 것'을 선택했습니다.
복잡한 공간 연산을 DB 레벨에서 처리하기 위해 **PostgreSQL과 PostGIS**를 핵심 엔진으로 채택했습니다.
특히 지도 렌더링 성능을 위해 Node.js 서버가 직접 `ST_AsMVT` 함수를 활용해 바이너리 포맷인 프로토콜 버퍼(PBF)를 클라이언트에 스트리밍하는 구조를 설계했습니다.
이는 일반적인 REST API 대비 네트워크 페이로드를 획기적으로 줄이는 핵심 전략이었습니다.

---

## Slide 4: 핵심 기술 1 - On-the-Fly MVT 생성 (Backend Engineering)
**[화면 구성]**
- 코드 스니펫: `ST_AsMVT`, `ST_TileEnvelope`이 포함된 SQL 쿼리 (tiles.service.ts)
- 다이어그램: Client -> (Request x/y/z) -> Server -> (Spatial SQL) -> DB -> (Binary MVT) -> Client

**[대본]**
가장 큰 기술적 성취는 **동적 MVT(Mapbox Vector Tile) 생성 시스템**입니다.
미리 만들어둔 타일 이미지를 서빙하는 것이 아니라, 사용자의 요청이 들어올 때마다 DB에서 실시간으로 공간 데이터를 조회하여 벡터 타일을 생성합니다.
이 과정에서 `ST_TileEnvelope`로 뷰포트 영역을 계산하고, 줌 레벨에 따라 보여줄 데이터의 밀도를 조절하는 로직(`random() < 0.3` 등)을 쿼리에 직접 녹여내어,
데이터의 최신성을 유지하면서도 쿼리 성능을 밀리초(ms) 단위로 유지했습니다.

---

## Slide 5: 핵심 기술 2 - 복합 공간 쿼리 최적화 (Query Tuning)
**[화면 구성]**
- Before/After 쿼리 비교 (Subquery vs JOIN)
- 실행 계획(EXPLAIN ANALYZE) 결과 요약 그래프 (속도 향상 수치)
- 키워드: "Query Optimization", "Intersection Logic"

**[대본]**
지도 위에서 각 행정동별 뉴스, 구독자, 콘텐츠 수를 집계하여 보여주는 기능은 초기 구현 시 심각한 성능 저하를 겪었습니다.
행마다 서브쿼리가 실행되는(N+1 문제) 구조였기 때문입니다.
이를 해결하기 위해 **CTE(Common Table Expressions)와 JOIN 기반의 집계 쿼리**로 전면 리팩토링했습니다.
`tiles.service.ts` 내의 복잡한 로직을 단일 쿼리로 통합하고, 공간 인덱스(`GIST`)를 적극 활용하여 수백 개의 행정동 데이터를 집계하는 시간을 90% 이상 단축시켰습니다.

---

## Slide 6: 핵심 기술 3 - 논블로킹 대용량 로그 시스템 (Async Architecture)
**[화면 구성]**
- 다이어그램: Main Thread -> `setImmediate` -> Data Compression -> DB Upsert
- 코드 스니펫: `saveApiLogSingle` 메서드의 `Promise.race` 및 압축 로직

**[대본]**
사용자의 활동 로그와 공간 분석 결과는 매우 방대한 데이터입니다.
이 데이터를 저장하느라 API 응답이 늦어지는 것을 방지하기 위해 **Node.js의 이벤트 루프 특성을 활용한 논블로킹 시스템**을 구현했습니다.
`setImmediate`를 사용하여 로그 저장을 백그라운드 태스크로 미루고, 메모리 효율을 위해 데이터를 압축하여 저장했습니다.
또한 `Promise.race`를 이용해 타임아웃을 강제하여, 로그 저장 지연이 절대 메인 비즈니스 로직에 영향을 주지 않도록 안전장치를 마련했습니다.

---

## Slide 7: 아키텍처 - 모듈러 모놀리스 (Modular Monolith)
**[화면 구성]**
- 프로젝트 폴더 구조 (Domain별 분리)
- 도메인 간 의존성 그래프
- 장점: 단일 배포의 편리함 + MSA로의 전환 용이성

**[대본]**
1인 개발의 생산성을 위해 **모놀리식 아키텍처**를 선택했지만, 스파게티 코드가 되지 않도록 엄격한 **모듈러 모놀리스** 구조를 지향했습니다.
`Analyze`, `Tiles`, `Authentication` 등 도메인별로 모듈을 철저히 분리하여 응집도를 높였습니다.
이는 혼자서 수십 개의 기능을 개발하면서도 코드의 복잡도를 제어할 수 있었던 비결이며, 추후 특정 모듈(예: 지도 타일 서버)만 별도로 분리하여 스케일 아웃할 수 있는 확장성까지 고려한 설계입니다.

---

## Slide 8: 트러블 슈팅 - "메모리 누수와 무한 루프를 잡아라" (Troubleshooting)
**[화면 구성]**
- 이슈: 소셜 로그인 시 탭이 무한히 열리는 버그 & MVT 생성 시 힙 메모리 증가
- 해결: `App State` 리스너 최적화 & Node.js GC 튜닝 옵션 (`--optimize-for-size`)

**[대본]**
개발 중 가장 아찔했던 순간은 소셜 로그인 시 브라우저 탭이 무한히 증식하는 '무한 루프' 버그였습니다.
원인은 `AppState` 변경 감지 시마다 토큰 체크 로직이 중복 실행되는 레이스 컨디션이었습니다. 이를 **플래그 변수와 상태 머신 패턴**을 도입하여 단 한 번만 실행되도록 엄격히 제어해 해결했습니다.
또한, 대량의 벡터 타일 생성 시 Node.js 힙 메모리가 요동치는 문제를 `package.json`의 실행 스크립트에 `--max-old-space-size` 설정과 GC 옵션을 튜닝하여 안정적인 메모리 사용량을 확보했습니다.

---

## Slide 9: 성과 및 데이터 (Impact)
**[화면 구성]**
- GitHub Commit Graph (200+ commits)
- 앱 스토어 등록 스크린샷
- 주요 성능 지표: 타일 로딩 속도 < 100ms, 공간 쿼리 최적화 90%

**[대본]**
이 프로젝트는 단순한 기능 구현을 넘어, **'극한의 성능 최적화'**를 경험한 과정이었습니다.
200회 이상의 커밋, 수만 라인의 코드 속에는 MVT 바이너리 처리부터 비동기 시스템 설계까지 현업 수준의 고민이 담겨 있습니다.
백엔드 엔지니어로서 데이터베이스부터 인프라, 그리고 클라이언트 최적화까지 전체 데이터 파이프라인을 장악해 본 값진 경험이었습니다.

---

## Slide 10: 마무리 (Closing)
**[화면 구성]**
- Github: [https://github.com/insungsong](https://github.com/insungsong)
- App Info: [https://insungsong.github.io/eodisani-docs/marketing.html](https://insungsong.github.io/eodisani-docs/marketing.html)
- 문구: "Deep Tech, Real Impact."

**[대본]**
'어디사니'는 제가 가진 기술적 역량의 정수입니다.
복잡한 공간 데이터를 효율적으로 다루고, 사용자에게 최고의 퍼포먼스를 제공하기 위해 치열하게 고민했습니다.
어려운 기술적 난제를 피하지 않고 정면으로 돌파하는 엔지니어, **송인성**이었습니다.
경청해 주셔서 감사합니다.
