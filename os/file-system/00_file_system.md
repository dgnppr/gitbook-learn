# 파일 시스템

## 파일이란

- 컴퓨터는 데이터를 저장 매체에 저장하는데, OS는 데이터를 편리하게 사용하기 위해 파일이라는 개념으로 관리한다.
- 파일은 OS에 의해서 물리 장치(하드디스크, CD-ROM 등)에 저장되고, 관리된다.
- 일반적으로 파일은 프로그램과 데이터를 의미한다.
- 파일은 `OS`마다 다른 속성을 갖지만 일반적으로 파일은 이름, 크기, 유형, 위치, 생성 시간, 접근 권한 등의 속성을 갖는다.

<br><br><br>

## 파일 시스템이란

- 파일 시스템은 디스크에서 쉽게 파일을 저장하고, 찾고, 인출할 수 있게 함으로써 보다 효울적으로 파일을 다룰 수 있게 해준다. 
- 파일 시스템은 크게 데이터를 저장하는 **파일** 그리고 파일을 관리하는 **디렉토리**로 구성된다.

<br><br><br>

## 디렉토리 구조

- 디렉토리는 파일 이름을 디렉토리의 파일 컨트롤 블록으로 매핑하는 테이블이다.
- 디렉토리는 여러 구조를 가질 수 있고 싱글 레벨부터 멀티 레벨까지 다양하게 구성할 수 있다.

![Screenshot 2024-01-04 at 00 12 39](https://github.com/dragonappear/gitbook-learn/assets/89398909/c0272731-b3bc-47c0-9b61-c498335d29e0)
<img src="https://github.com/dragonappear/gitbook-learn/assets/89398909/a7aabbad-7327-45e1-be62-37e57a2acab1">

<br>

```Bash
javac *.java
```

자바 파일을 컴파일을 하는 명령어인데, `javac` 라는 파일은 디스크에 컴파일과 관련된 프로그램을 매핑되어 있고, `javac`를 호출하면서 그 프로그램을 실행시키는 것이다.

새로운 프로그램을 설치하고 터미널에서 실행했을때(ex: `java` 등) 환경 변수 PATH 에 등록하는 이유도 파일 위치를 매핑하기 위해서이다.

<br><br><br>

## 파일 할당

- 디스크는 물리적으로 섹터로 구성되어 있고, 섹터는 블록으로 구성되어 있다. 파일은 각 블록에 할당된다.
- 메모리 할당 이슈가 있는 것처럼, 파일도 블록에 어떻게 할당한 것인지에 대한 이슈가 있다.

<br>

### Contiguous Allocation

<img src="https://github.com/dragonappear/gitbook-learn/assets/89398909/96d3ec8a-332a-41a9-ab6a-faabd903d60b" height="400">

- 파일을 연속적인 섹터에 할당하는 방법
- 연속적인 메모리 할당과 비슷한 방식으로 파일을 할당한다.
- 파일을 편하게 블록에 할당할 수 있지만
- 파일을 삭제하면 빈 공간이 생기고, 이 빈 공간을 활용하기 위해서는 파일을 압축해야 한다.
- 또한 외부 단편화가 발생할 수 있다.

<br>

### Linked Allocation

<img src="https://github.com/dragonappear/gitbook-learn/assets/89398909/6871952c-3672-46ea-bf5a-f13f65111f98">

- 파일을 불연속적으로 블록에 할당하고, 각 블록을 순차적으로 링크드 리스트로 연결하는 방법
- 외부 단편화와 압축 이슈는 해결할 수 있지만, 파일의 i번째 블록을 찾기 위해서는 첫 번째 블록부터 순차적으로 탐색해야 한다는 단점이 있다.
- 또한, 파일의 중간 블록이 고장나면 파일의 나머지 블록에 접근할 수 없다.

<br>

### FAT(File Allocation Table)

<img src="https://github.com/dragonappear/gitbook-learn/assets/89398909/ce43981c-d8ea-46d0-9d70-50ed081c47ec">

- 파일을 링크드 할당 방식으로 할당하면서, 각 블록의 위치를 FAT 테이블에 저장하는 방식
- FAT 테이블은 디스크의 한 섹터에 저장되고, 시작 블록 번호를 인덱스로 하여 블록의 위치를 찾을 수 있다.
- 링크드 방식과 동일하게, 중간 블록이 고장나면 파일의 나머지 블록에 접근할 수 없다.

<br>

### Indexed Allocation

<img src="https://github.com/dragonappear/gitbook-learn/assets/89398909/8169e425-e1d5-4ffc-8110-27fc0331e5fa" height="400">

- 파일의 모든 블록의 위치를 인덱스 블록에 저장하는 방식
- 인덱스 블록은 디스크의 한 섹터에 저장되고, 파일의 i번째 블록을 찾기 위해서는 인덱스 블록에 접근하여 i번째 블록의 위치를 찾을 수 있다.

<br><br><br>

## 파일 시스템 구조

파일 시스템은 계층 구조로 구성되어 있다.

<img src="https://github.com/dragonappear/gitbook-learn/assets/89398909/f45c74e1-f0a3-4fca-af87-5629482992e7" height="400">

- `devices`
  - 장치(하드디스크, CD-ROM 등)를 관리하는 계층
- `I/O control`
  - 메모리와 디스크 시스템 사이에 정보 전송을 담당하는 계층
- `Basic file system`
  - 적절한 디바이스 드라이버에게 디스크 상의 물리 블록을 읽고 쓰도록 명령을 내리는 계층
- `File organization module`
  - 파일의 논리 블록과 물리 블록을 매핑하는 계층
- `Logical file system`
  - 메타 데이터 정보를 관리하는 계층
  - 메타데이터는 파일의 내용 자체인 데이터를 제외한 모든 파일시스템 구조를 말한다 (파일의 이름, 유형, 크기, 생성 시간, 접근 권한 등의 속성)
  - `Logical file system` 계층은 `inode`를 사용하여 메타 데이터를 관리한다.

<br><br><br>

## inode

- UNIX 및 유닉스 계열 운영 시스템에서 사용되는 데이터 구조로 파일 시스템 내의 각 파일과 디렉토리에 대한 메타데이터를 저장한다.
- `inode`에는 다음과 같은 정보가 포함된다.
    - 파일의 소유자와 그룹 정보, 파일 유형, 파일 권한, 파일의 타임 스탬프, 파일 크기와 같은 메타 데이터
- `inode`는 파일의 데이터 블록에 대한 포인터를 가지고 있다.
- 리눅스에서 사용하는 파일 시스템인 `ext`에 대한 `inode`는 아래와 같다.

<img src="https://github.com/dragonappear/gitbook-learn/assets/89398909/b9ec18e2-7947-438c-84bb-1dff7b31cb07" height="400">

<br><br>

파일 시스템에서의 블록에 대해서 자세하게 보면 아래 그림과 같이 구성되어 있다.

<img src="https://github.com/dragonappear/gitbook-learn/assets/89398909/db5a9a1d-f25a-43e6-8db7-7a7f8ef91f4e" height="400">

파일 시스템에서의 블록은 `super block`, 그 파일의 메타 데이터는 `inode block`, 그리고 실제 데이터를 `data block` 으로 구성됨을 볼 수 있다.

이렇게 계층화를 하여 파일 시스템을 구성하는 이유는 아래 `inode` 구조를 보면 알 수 있다.

<br><br><br>

<img src="https://github.com/dragonappear/gitbook-learn/assets/89398909/1e5f5125-22c4-4690-8ce4-c6f028d03f4f" height="400">

- `inode block` 구성을 보면 `data block`의 위치를 가리키는 포인터가 존재한다. 
  - 이 포인터는 `direct pointer`, `indirect pointer`, `double indirect pointer` 로 구성되어 있다.
    - **핵심은 블록의 크기가 최대 4KB 이다.** 
    - 블록의 크기가 제한되어있기 때문에 `direct pointer`, `indirect pointer`, `double indirect pointer`로 나누어 확장하여 관리한다 

- `direct pointer`
  - 파일의 데이터 블록을 가리키는 포인터
  - `inode`에는 12개의 `direct block`이 존재한다.
  - `direct block`는 파일의 데이터 블록을 가리키는 포인터이기 때문에, 파일의 크기가 4KB 이하일 경우에는 `direct pointer`만 사용하게 된다.
- `indirect pointer`
  - `indirect pointer`는 데이터 블록을 가리키는 포인터이기 때문에, 파일의 크기가 4KB를 넘어서는 경우에 사용된다.
- `double indirect pointer`
  - `double indirect pointer`는 `indirect pointer`를 가리키는 포인터이기 때문에, 파일의 크기가 4MB를 넘어서는 경우에 사용된다.

**이렇게 블록 크기를 넘어서면 `indirect pointer`를 사용하고, `indirect pointer`의 개수가 부족하면 `double indirect pointer`를 사용하는 방식으로 파일의 크기를 확장할 수 있다.**




<br><br><br>
  
## Ref

- https://velog.io/@redgem92/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C-%ED%8C%8C%EC%9D%BC-%EC%8B%9C%EC%8A%A4%ED%85%9C-inode-%EB%B0%A9%EC%8B%9D%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC