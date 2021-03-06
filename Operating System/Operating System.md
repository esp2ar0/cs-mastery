# Operating System

### 목차

- [1장 페이징(Paging)](#1.-페이징(paging))
  - [1.1 메모리 주소 매핑](#1.1-메모리-주소-매핑)
    - [1.1.1 1단계 페이징](#1.1.1-1단계-페이징)
    - [1.1.2 계층적 페이징](#1.1.2-계층적-페이징)

---

## 1. 페이징(Paging)

페이징은 논리 주소 공간과 물리 주소 공간을 나누어 다루는 메모리 관리 기법이다.

즉, 가상 주소 공간을 만들어 이전의 문제들(스왑 아웃 시 디스크에서 연속적인 공간을 찾는 문제, 외부 단편화, 압축 등과 같은 기법을 사용함에 있어서 속도의 문제..)을 해결한다.



프로세스마다 보조 기억 장치로부터 주 기억 장치(메인 메모리)로 메모리를 할당하게 된다.

이 때, 연속된 메모리 공간을 할당하기 위해 오버헤드가 발생한다.

페이징 기법은 물리 주소 공간에서 연속적인 공간에 모여 있어야 한다는 제약을 없앤다.

물리 주소 공간에서는 연속적이지 않지만 가상 주소 공간에서 연속적으로 할당해 다루는 것이다.



물리 메모리는 프레임(frame)이라고 불리는 고정 크기 블록,

논리 메모리는 페이지(page)라고 불리는 고정 크기 블록이 있다.





![1](https://user-images.githubusercontent.com/20038567/47486911-5ba6c300-d87c-11e8-9a56-67e83b5a8047.png)

(사진 출처 :  http://kdy-study.tistory.com/14)



논리적(가상) 메모리에 페이지를 연속적으로 할당하고, 물리적 메모리에서 페이지가 들어있는 프레임은 흩어져 있게 된다.

페이지 테이블을 통해 페이지와 프레임을 매핑한다.





## 1.1 메모리 주소 매핑



x86 아키텍처 환경에서 논리 주소가 물리 주소로 어떻게 변환되는지 알아보자.

> 32비트 프로세스임을 뜻한다.

32비트 프로세스에서는 0x00000000 ~ 0xFFFFFFFF의 주소 공간(4GB : 2^{32})을 가진다.

그리고 페이지의 크기는 4KB(2^{12})이다.

> 하나의 페이지는 x86과 amd64에서는 4KB, ia64에서는 8KB의 크기를 가진다.
>
> 논리 주소 공간의 크기가 2의 m승이고, 페이지가 2의 n승이라면 논리주소(logical address)의 상위 m-n Bit는 페이지 번호를 하위 n비트는 페이지 변위(offset)을 나타낸다.





### 1.1.1 1단계 페이징

기본적으로 위에서 알아본 페이징은 1단계 페이징이다.

![2](https://user-images.githubusercontent.com/20038567/47486921-606b7700-d87c-11e8-9681-538a6796268b.PNG)



페이지 테이블의 항목(레코드)을 페이지 테이블 엔트리라 한다.

엔트리의 크기는 32비트 프로세스에서 일반적으로 4B(32비트)이며, 프레임 주소와 플래그 비트를 가진다.

프레임 주소는 엔트리 크기인 32비트에서 페이지 크기인 12비트를 뺀 20비트만큼 차지한다.

즉, 상위 20비트는 프레임의 주소, 하위 12비트는 플래그 비트이다.

엔트리에서 상위 20비트의 프레임 주소를 구하고, 하위 12비트는 페이지 변위(offset)를 더하면 32비트의 물리 주소를 구할 수 있다.



페이지 크기, 페이지 테이블 크기, 엔트리 개수, 엔트리 크기 등의 개념을 잘 잡아야 한다.

위와 같은 환경에서는 다음과 같다.

- 페이지 크기 = 4KB
- 엔트리 개수 = 1048576개(2^{20})
- 엔트리 크기 = 4B
- 페이지 테이블 크기 = 4MB (엔트리 개수 * 엔트리 크기)



1단계 페이징에서 논리 주소를 물리 주소로 매핑하기

1) 레지스터로 페이지 테이블의 시작 주소를 얻는다.

2) 시작 주소에 페이지 번호를 더해 페이지 테이블에 접근한다.

3) 페이지 테이블 엔트리에서 얻은 프레임 주소에 페이지 변위를 더해 최종 물리 주소를 구한다.



이 경우, 페이지 테이블을 저장하기 위해서만 4MB의 물리공간이 필요하다.

페이지 테이블의 크기가 커질 수록 주 메모리에서 연속적으로 할당되기를 기대할 수는 없다.

따라서 계층적 페이징이 등장하게 됐다.





### 1.1.2 계층적 페이징

계층적 페이징은 페이지 테이블을 위한 페이지 테이블을 만드는 것이다.

즉, 페이지 테이블 자체가 다시 페이징 되는 것이다.(2단계 페이징 기법)

20비트의 페이지 테이블을 각 10비트씩 나눠보자.



![3](https://user-images.githubusercontent.com/20038567/47486922-606b7700-d87c-11e8-9e8e-a52bf32d4098.PNG)



이 경우 페이지 크기, 페이지 테이블 크기, 엔트리 개수, 엔트리 크기는 다음과 같다.

- 페이지 크기 = 4KB
- 엔트리 크기 = 4B
- 첫 번째 페이지 테이블 엔트리 개수 = 1024개
- 첫 번째 페이지 테이블 크기 = 4KB
- 두 번째 페이지 테이블 엔트리 개수 = 1024개
- 두 번째 페이지 테이블 크기 = 4KB



2단계 페이징 기법에서 첫 번째와 두 번째 페이지 테이블을 각각 페이지 디렉토리, 페이지 테이블이라 부른다.



2단계 페이징에서 논리 주소를 물리 주소로 매핑하기

1) 레지스터에서 페이지 디렉토리의 시작 주소를 알아낸다.

2) 시작 주소에 페이지 디렉토리 인덱스를 더해 페이지 디렉토리에 접근한다.

3) 페이지 디렉토리 엔트리에서 페이지 테이블 시작주소를 알아낸다.

4) 페이지 테이블 시작주소에 페이지 테이블 인덱스를 더해 페이지 테이블에 접근한다.

5) 페이지 테이블 엔트리에서 프레임 주소를 알아낸다.

6) 프레임 주소에 페이지 변위를 더해 최종 물리 주소를 구한다.



![4](https://user-images.githubusercontent.com/20038567/47486923-606b7700-d87c-11e8-992e-59c19e7575e8.PNG)



그렇다면, 8MB의 메모리가 필요할 때

각 페이지 테이블은 몇 번 접근해야 할까?



페이지 크기가 4KB이기 때문에 두 번째 페이지 테이블은 2048(2^{20} = 8MB/4KB)번의 접근이 필요하다.

두 번째 페이지 테이블을 2048번 접근하기 위해서는 첫 번째 페이지 테이블을 2번 접근해야 한다.







3단계 페이징 기법으로 확장 할 경우 페이지 테이블 할당을 위해 페이지 크기가 줄어들게 될 것이다.

해당 경우는 개인적으로 생각해보기 바란다.