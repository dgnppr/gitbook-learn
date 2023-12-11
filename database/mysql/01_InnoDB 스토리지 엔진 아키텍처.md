# InnoDB 스토리지 엔진

## 아키텍처

![https://dev.mysql.com/doc/refman/8.0/en/innodb-architecture.html](https://dev.mysql.com/doc/refman/8.0/en/images/innodb-architecture-8-0.png)

<hr>

## 특징

### [PK에 의한 클러스터링](https://dev.mysql.com/doc/refman/8.0/en/innodb-index-types.html)

- InnoDB의 모든 테이블은 기본적으로 PK를 기준으로 클러스터링된다. 즉, PK 값 순서대로 디스크에 저장된다.
- PK(클러스터링) 인덱스가 자동 생성된다.
- 모든 세컨더리 인덱스는 레코드 주소 대신 PK 값을 논리적인 주소로 사용한다. 
  - PK를 통해서만 레코드에 접근 가능
  - PK를 통한 레인지 스캔 매우 빠름
- 클러스터링 때문에 쓰기 성능 저하

> `MyISAM` 스토리지 엔진에서는 클러스터링 키를 지원하지 않는다. PK는 유니크 제약을 가진 세컨더리 인덱스일 뿐이다.
> <br>
> `MyISAM` 테이블의 프라이머리키를 포함한 모든 인덱스는 물리적인 레코드의 주소 값(`ROWID`)를 가진다.

<hr>

### [외래키 지원](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html)

- InnoDB 스토리지 엔진 레벨에서 지원하는 기능으로 `MyISAM`이나 `MEMORY` 테이블에서는 사용할 수 없음
- **부모, 자식 테이블 모두 해당 컬럼에 대한 인덱스 생성이 필요하고, 변경 시에는 반드시 부모, 자식 테이블에 데이터가 있는지 체크하는 작업이 필요하므로 잠금이 여러 테이블로 전파되고, 그로 인해 데드락이 발생할 때가 많으므로 개발할 때 외래 키의 존재에 주의하는 것이 좋다.**

```SQL
SET foreign_key_checks = OFF;

-- // 작업 실행

SET foreign_key_checks = ON;
```

- `foreign_key_checks` 시스템 변수를 `OFF`로 설정하면 외래 키 관계애 대한 체크 작업을 일시적으로 멈출 수 있음
  - 멈추면 `ON DELETE CASCADE`와 `ON UPDATE CASCADE`도 무시하게 된다.  
  - 일시적으로 해제했다고 테이블 간의 관계가 깨진 상태로 그대로 유지해도 된다는 것을 의미하지 않음. 꼭 데이터 일관성을 맞춰줘야 한다.
- `foreign_key_checks` 시스템 변수는 적용 범위를 글로벌과 세션 모두 설정 가능한 변수이다. 작업을 할 때는 현재 작업을 실행하는 세션에서만 외래 키 체크 기능을 멈추게 해야한다.
  - `SESSION` 키워드를 명시하지 않으면 자동으로 현재 세션의 설정만 변경한다.


<hr>

### [MVCC](https://dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html)

- `MVCC`는 `InnoDB` 스토리지 엔진의 특징 중 하나로, `MVCC`를 사용하면 `InnoDB` 스토리지 엔진은 트랜잭션의 격리 수준을 위해 잠금을 사용하지 않는다.
  - 즉, 잠금을 사용하지 않고하나의 레코드에 대해 여러 개의 버전이 동시에 관리될 수 있다는 것을 의미한다.
- `InnoDB`는 [언두 로그](https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-tablespaces.html)를 사용해 이 기능을 구현한다.

<br>

#### 언두 로그 영역을 활용한 MVCC

![https://www.youtube.com/watch?v=vQFGBZemJLQ](https://github.com/dragonappear/learn/assets/89398909/3a14f761-519e-4770-b22a-1a6928428535)

`UPDATE` 쿼리가 실행되면 커밋 실행 여부와 관계없이 InnoDB 버퍼풀은 새로운 값으로 업데이트 된다. 디스크의 데이터 파일에는 체크포인트나 InnoDB의 Write 스레드에 의해 새로운 값으로 업데이트 되었을수도 있고 아닐 수도 있다.

아직 커밋이나 롤백이 되지 않은 상태에서 다른 사용자가 작업 중인 레코드를 조회하면 어떻게 데이터가 조회될까? 이 질문의 답은 MySQL 서버의 시스템 변수(`transaction_isolation`)에 설정된 격리 수준에 따라 달라진다.
 
- `READ_UNCOMMITTED`: InnoDB 버퍼 풀이 현재 가지고 있는 변경된 데이터를 읽어서 반환. 즉, 커밋됐든 아니든 변경된 상태의 데이터를 반환한다
- `READ_COMMITTED`나 그 이상의 격리 수준인 경우 : 아직 커밋되지 않았기 때문에 InnoDB 버퍼 풀이나 데이터 파일에 있는 내용 대신 변경되기 이전의 내용을 보관하고 있는 언두 영역의 데이터를 반환

<br>

#### 언두 영역 데이터 삭제는 언제될까?

- 롤백을 실행하면 `InnoDB`는 언두 영역에 있는 백업된 데이터를 InnoDB 버퍼풀로 다시 복구하고, 언두 영역의 내용을 삭제해버린다. 
- 커밋이 되었다고 해서 언두 영역의 백업 데이터가 항상 바로 삭제되는 것은 아니다. 
- 언두 영역을 필요로 하는 트랜잭션이 더는 없을 때 비로소 삭제된다.

> 트랜잭션이 길어지면 언두에서 관리하는 예전 데이터가 삭제되지 못하고 오랫동안 관리되어야 하며, 언두 영역이 저장되는 시스템 테이블스페이스의 공간이 많이 늘어나는 상황이 발생할 수 있다.

<hr>

## Ref

- https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html
- https://www.youtube.com/watch?v=vQFGBZemJLQ