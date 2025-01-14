---
title: PostgreSQL Indexing
description: >
author: Jongkwan Lee
date: 2025-01-14 20:00:00 +0900
categories: [PostgreSQL]
tags: [PostgreSQL]
pin: false
math: true

---

# PostGresql과 MySQL의 인덱싱 차이점
## 1. 지원 인덱스 타입 비교

### PostgreSQL

1. **B-Tree**
  - 가장 일반적으로 사용되는 인덱스. 비교 연산(<, <=, =, >=, >)이 주가 되는 데이터에 최적화되어 있습니다.

2. **Hash**
  - 특정 해시 함수를 통해 빠르게 동등 연산(=) 비교를 수행할 수 있도록 지원합니다.
  - PostgreSQL 10부터 WAL(Write-Ahead Logging)을 지원하면서, 장애 복구 시에도 Hash 인덱스가 유실되지 않도록 개선되었습니다.

3. **GIN(Generalized Inverted Index)**
  - 텍스트 검색, JSONB, 배열 등의 “내부 요소”를 색인화하기 위한 인덱스.
  - **Full Text Search**, **JSONB 키 탐색**, **배열 검색** 등에 효과적입니다.

4. **GiST(Generalized Search Tree)**
  - R-Tree, k-Nearest Neighbor(KNN) 검색 등 다양한 검색 방식을 추상화하여 구현할 수 있는 인덱스 구조.
  - 지리 정보(PostGIS), 범위 타입(Range Type) 등에서 활용됩니다.

5. **BRIN(Block Range Index)**
  - 대용량 테이블에서 '범위' 단위로 최소/최대값 등을 기록하여 검색 시 범위를 빠르게 좁히도록 해주는 인덱스.
  - 시계열이나 순차적으로 증가하는 ID, 타임스탬프 등에 매우 효율적입니다.

6. **SP-GiST(Space-Partitioned GiST)**
  - GiST의 변형으로, 트라이(trie) 기반 등 다양한 공간 분할 알고리즘을 활용할 수 있습니다.

### MySQL (InnoDB 기준)

1. **B-Tree**
  - MySQL(InnoDB)의 기본 인덱스 타입이며, 대부분의 인덱스가 B-Tree 구조를 사용합니다.
  - VARCHAR 같은 긴 텍스트 컬럼은 인덱스 전체를 사용하는 대신 “Prefix 인덱스(접두사 인덱스)” 설정으로 일부만 인덱싱하는 경우가 흔합니다.

2. **Hash**
  - InnoDB 엔진 자체적으로는 Hash 인덱스를 사용자가 직접 생성할 수 없고, **Adaptive Hash Index**(AHI)라는 내부 최적화 기능만 존재합니다.
  - MEMORY(HEAP) 테이블 엔진에서만 해시 인덱스를 설정할 수 있지만, 실무에서 많이 사용되지는 않습니다.

3. **Full-text 인덱스**
  - InnoDB에서 **FULLTEXT** 인덱스를 지원하며, 텍스트 검색을 위해 역색인(Inverted Index) 기법을 사용합니다.
  - 다만 PostgreSQL의 GIN/GiST를 활용한 Full Text Search에 비해 문법과 동작 방식이 단순한 편입니다.

4. **RTREE**
  - MySQL 8.0 이상에서 GIS 기능을 사용할 때 R-Tree 기반 인덱스를 사용할 수 있지만, 일반 컬럼용은 아님.

---

## 2. 부분 인덱스(Partial Index) 및 표현식 인덱스(Expression Index)

### PostgreSQL

- **Partial Index**
  - `CREATE INDEX idx_name ON table(column) WHERE some_condition;`
  - 인덱스가 필요한 “일부 데이터만” 인덱싱하여, 디스크 사용량과 불필요한 인덱스 유지 비용을 줄일 수 있습니다.
  - 예: status='ACTIVE'인 행만 인덱스하는 경우 등.

- **Expression Index**
  - `CREATE INDEX idx_lower_name ON table((LOWER(name)));`
  - 특정 함수나 표현식 결과를 기반으로 인덱스를 만들 수 있어, 질의 최적화에 매우 유용합니다.
  - 예: 문자열을 대소문자 구분 없이 검색해야 하는 경우, JSON 키의 특정 요소만 추출하여 인덱싱하는 경우 등.

### MySQL

- **Partial Index**
  - MySQL에서 “부분 인덱스”라는 개념은 PostgreSQL과 달리 “일부 행(row)”을 대상으로 하는 것이 아니라, “일부 컬럼(prefix)”을 대상으로 하는 것을 가리키는 경우가 많습니다. (접두사 인덱스)
  - 즉, `CREATE INDEX idx_name ON table(column(10));` 이런 식으로 문자열 앞부분만 인덱싱하여 인덱스 크기를 줄이는 방법을 말합니다.
  - MySQL 8.0에서 **Invisibile Index** 같은 기능이 추가됐지만, PostgreSQL의 Partial Index 같은 “조건 기반” 부분 인덱스는 공식적으로 지원하지 않습니다.

- **Expression Index**
  - MySQL 8.0.13부터 “함수 기반 인덱스(Functional Index)”가 도입되어 표현식 인덱스를 제한적으로 지원합니다.
  - 하지만 PostgreSQL의 Expression Index만큼 자유도가 높지는 않으며, 제약 사항이 더 많은 편입니다.
  - 또한 MySQL 8.0 미만 버전에서는 함수 기반 인덱스를 직접 구현할 수 없어, 컬럼을 별도로 두어 트리거로 동기화하는 방식으로 우회해야 했습니다.

---

## 3. Full Text Search 및 JSON 인덱싱

### PostgreSQL

- **Full Text Search**
  - 내장된 텍스트 검색 기능(TSearch2 기반)과 GIN/GiST 인덱스를 활용합니다.
  - 다양한 언어별 토큰화, 불용어 처리, 랭크 매기기 등이 가능하고, 쿼리 문법도 유연합니다.

- **JSONB 인덱싱**
  - JSONB 타입으로 저장된 반정형 데이터를 GIN 인덱스 등으로 색인화할 수 있습니다.
  - `jsonb_path_ops` 같은 전용 오퍼레이터 클래스를 사용하면 특정 키/값 조회 성능을 크게 높일 수 있습니다.

### MySQL

- **Full Text Search**
  - InnoDB에서 FULLTEXT 인덱스를 만들 수 있으나, PostgreSQL GIN 기반 텍스트 검색보다 검색 기능이나 구문 분석 면에서 제약이 많은 편입니다.
  - 기본적으로 자연어 검색, Boolean 모드, Query Expansion 모드를 지원하지만, 다국어 형태소 분석 등의 세밀한 처리는 직접 구현해야 하는 경우가 많습니다.

- **JSON 인덱싱**
  - MySQL 5.7부터 JSON 타입을 지원하며, 8.0부터는 **함수 기반 인덱스**를 사용해 JSON의 특정 경로(JSON path)에 대한 인덱스를 구성할 수 있습니다.
  - 다만 PostgreSQL의 GIN처럼 JSON 전체 구조를 역색인화할 수 있는 강력한 기능은 상대적으로 부족합니다.

---

## 4. 동시성(Concurrency) 및 인덱스 관리

### PostgreSQL

1. **CREATE INDEX CONCURRENTLY**
  - 인덱스를 생성하는 동안 테이블에 쓰기 잠금(Write Lock)을 걸지 않고, 읽기/쓰기도 어느 정도 병행할 수 있게 하는 기능입니다.
  - 대용량 테이블에서도 서비스 중단 없이 인덱스를 추가할 수 있어 운영 환경에서 유용합니다.

2. **인덱스 유지보수(아날라이즈, REINDEX 등)**
  - `ANALYZE` 명령으로 통계를 업데이트하고, 옵티마이저가 더 정확한 실행 계획을 세울 수 있도록 돕습니다.
  - 오래된 인덱스가 부풀어올랐다면 `REINDEX` 명령으로 다시 생성하여 최적화할 수 있습니다.

### MySQL

1. **온라인 DDL(Online DDL)**
  - InnoDB에서 여러 종류의 온라인 DDL(Operation)을 지원하여, 테이블 읽기/쓰기를 중단 없이 인덱스를 생성하거나 수정할 수 있습니다.
  - 단, 일부 작업은 배타적 잠금이 필요한 경우도 있어, 실제 운영 환경에서는 제한 사항을 주의해야 합니다.

2. **통계 정보 수집**
  - MySQL도 자동/수동 `ANALYZE TABLE`을 통해 통계 정보를 재수집할 수 있으며, 옵티마이저가 이를 활용합니다.
  - InnoDB의 통계는 자동 수집되는 부분이 있지만, 때때로 정확하지 않거나 빈번히 변동될 수 있어 성능 문제가 발생하기도 합니다.

---

## 5. 성능 및 최적화 관점

1. **B-Tree 인덱스 성능**
  - 두 DBMS 모두 B-Tree 인덱스가 메인이고, 일반적인 범위 쿼리와 동등 조건 쿼리에 강점이 있습니다.
  - 다만 PostgreSQL은 Partial Index, Expression Index 등을 통해 범위를 줄이거나 특정 조건만 효율적으로 인덱싱함으로써 추가 최적화를 시도할 수 있습니다.

2. **대규모 텍스트 / JSON 검색**
  - PostgreSQL은 GIN/GiST를 통한 **고급 검색**(Full Text Search, JSONB indexing)에 강점이 있으며, 복잡한 조건 쿼리에서도 옵티마이저가 잘 활용합니다.
  - MySQL도 FULLTEXT 인덱스나 함수 기반 인덱스로 JSON 경로를 부분적으로 최적화할 수 있지만, PostgreSQL에 비해 세밀함이나 확장성은 다소 떨어집니다.

3. **인덱스 크기와 관리 비용**
  - PostgreSQL의 Partial Index를 활용하면 불필요한 행까지 인덱싱할 필요가 없어 디스크 공간과 유지 비용을 절약할 수 있습니다.
  - MySQL은 긴 텍스트 컬럼에 접두사 인덱스를 자주 쓰기 때문에, 실제 검색에 따라서는 정확도가 떨어지거나 “Using index condition” 등으로 성능이 미묘해질 수 있습니다.

4. **옵티마이저의 실행 계획 차이**
  - PostgreSQL은 “코스트 기반 옵티마이저”가 다양한 인덱스 타입(GIN, GiST 등)을 종합적으로 고려하여 실행 계획을 세웁니다.
  - MySQL 역시 코스트 기반이지만, PostgreSQL에 비해 선택할 수 있는 인덱스 타입 종류가 적어 인덱스 활용 방식이 단순합니다.

---

## 6. 요약

- **인덱스 타입의 다양성**
  - PostgreSQL은 B-Tree 외에도 Hash, GIN, GiST, BRIN 등 다양한 인덱스 구조를 제공하여, 텍스트 검색·JSON·지리 정보·시계열 등 다양한 시나리오에서 고성능을 낼 수 있습니다.
  - MySQL은 기본적으로 B-Tree 위주이며, Full-text 인덱스와 R-Tree(GIS) 정도가 추가로 제공됩니다. (InnoDB 해시 인덱스는 내부 최적화용)

- **부분 인덱스 & 표현식 인덱스**
  - PostgreSQL은 Partial Index와 Expression Index를 오랫동안 지원해 왔으며, 이는 매우 강력한 최적화 수단입니다.
  - MySQL은 Partial Index(조건 기반) 개념이 없고, 8.0 이상에서 제한적인 함수 기반 인덱스를 지원합니다.

- **Full Text Search 및 JSON 인덱싱**
  - PostgreSQL의 GIN/GiST 인덱스는 고급 텍스트 검색, JSONB, 배열 검색 등을 폭넓게 지원합니다.
  - MySQL도 FULLTEXT 및 JSON 타입을 지원하지만, 기능 면에서 PostgreSQL만큼 세밀하거나 확장적이진 않습니다.

- **동시성 및 온라인 인덱스 생성**
  - PostgreSQL의 `CREATE INDEX CONCURRENTLY`, MySQL의 온라인 DDL 모두 운영 중단 없이 인덱스를 만들 수 있게 해주지만, 내부 동작 방식과 제한 사항이 서로 다릅니다.
  - 대용량 테이블에서 인덱스를 추가할 때는 각 DBMS가 허용하는 락(LOCK) 수준과 병행성 제약을 반드시 확인해야 합니다.

결과적으로 **PostgreSQL은 인덱싱 기능이 매우 다양**하고 **복잡한 시나리오에 대응하기 좋은 유연성**을 갖추고 있습니다. MySQL(InnoDB)은 **일반적인 B-Tree 인덱스로 충분한 범위 쿼리·동등 조건 쿼리를 빠르게 처리**하는 데 강점이 있지만, PostgreSQL의 Partial Index, GIN/GiST, Expression Index처럼 **고급 인덱스 기법**이 필요할 때는 기능 면에서 상대적으로 제약이 있습니다.

실제 운영 환경에서 “어떤 데이터 특성을 어떤 방식으로 자주 조회하는지”에 따라 인덱스 설계가 달라지므로, 두 DBMS 간 차이를 잘 이해하고 프로젝트 요구사항에 맞춰 선택·활용하는 것이 중요합니다.
