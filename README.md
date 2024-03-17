# null_shinsa

## 개요

- 해당 프로젝트는 무신사,29CM 같은 패션 이커머스 기반 프로젝트 입니다.
- 주문이나 재고와 같은 고민스럽고 실제 비즈니스 로직도 복잡하게 적용되야 하는 기능들을 좋은 방식으로 구현해보는 경험을 하고자 진행하게 되었습니다.

## 기술 스택

기본 스택
- JDK 17
- Spring Boot 2.8, JPA 
- RDS (Mysql)
- swagger
- Jenkins
- nginx
- 인메모리 로컬 캐시

추가 스택
- redis
- kafka
- docker
- ..

## 예상 아키텍처
<p align="center">
 <img src = "./92cm 구조도 (1).jpg">
</p>


## ERD 
<p align="center">
 <img src = "null_shinsa.png">
</p>

---

## 정책

### 구현 1순위 정책

1. 주문은 여러 상품이 묶여서 하나의주문이 된다.
    1. 메인주문 번호와 주문상품별 주문번호가 따로 필요할것.
2. 상품주문시 상품의 재고가 고려되야 한다. (동시성 이슈 가능)
3. 상품주문은 생성 당시의 데이터를 그대로 보관해야 한다(반정규화)
    1. 배송지
    2. 상품
    3. 결제정보
    4. 할인정보
4. 상품주문시 장바구니에서 해당 상품 데이터가 삭제되야한다.
5. 상품주문은 6가지 단계가 존재한다
    1. 입금대기 & 주문 접수 & 결제완료
    2. 출고처리중
    3. 출고완료
    4. 배송중
    5. 배송완료
    6. 구매확정
    7. 주문취소
6. 주문 수정, 취소는 입금대기 & 결제완료 상태 까지만 가능하다(그 이후는 반품으로 처리해야한다.)
    1. 출고 처리중은 셀러 입장에서 상품을 배송처리한후에 출고완료처리를 하는 경우 상품취소가 정상적으로 불가한 케이스도 존재할것
7. 상품주문은 대략적인 배송시작일, 배송완료일 정보가 존재해야 한다.
8. 주문, 상품주문은 주문의 변경을 저장하는 주문로그가 필요하다
9. 주문의 결제방법, 결제 결과를 저장해야겠지만 테이블만 만들어 놓을 예정.
10. 환불은 단계가 존재
    1. 환불 입금 대기 && 환불 접수 완료
    2. 환불 회수 요청
    3. 환불 회수 완료
    4. 환불 전달 완료
    5. 환불 처리 중
    6. 환불 완료
11. 환불 취소는 환불 회수 요청  단계까지만 가능
12. 교환은 단계가 존재
    1. 교환 입금 대기 && 교환 주문 접수
    2. 교환 회수 요청
    3. 교환 회수 완료
    4. 교환 전달 완료
    5. 교환 처리 중
    6. 교환 완료
    7. 교환 취소
13. 교환 취소는 교환 회수 요청 단계까지만 가능
14. 주문의 일정 단계 이상의 흐름은 외부 api(ex. 택배, 굿즈플로)에서 전달받는다고 가정
15. 결제가 생략되어 편의상 주문 단계중 입금대기, 접수 단계는 생략.
16. 쿠폰은 상품주문마다 적용 가능하다
17. 적립금은 주문시 사용가능하고 주문의 상품주문이 모두 취소되야만 환급이 가능하다
    1. a주문 - b상품주문, c상품주문, d상품주문
    2. c상품주문만 취소된 경우 적립금 환급 x
    3. b,d상품주문만 취소된 경우 적립금 환급 x
    4. b,c,d상품주문 모두 취소된 경우 적립금 환급
18. 적립금은 회원등급별로 적립율이 다르다
19. 회원할인은 회원등급별로 할인율이 다르다

---

## 기능구현 1순위 리스트

1. 로그인 (간단)
2. 로그아웃 (간단)
3. 상품등록
4. 브랜드등록
5. 주문등록
6. 주문 상태변경 api - 외부 업체 연동한다는 가정하에 단일 의뢰건 변경 & 다수 의뢰건 변경api 필요
    1. 주문 출고처리
    2. 주문 출고완료 → 배송시작
    3. 주문 배송완료
7. 주문 수정
8. 주문 취소
9. 주문삭제 (soft)
10. 환불등록
    1. 등록시 바로 환불 회수요청으로 진행
    2. 환불 방식 선택 필요
        1. 환불예정금액에서 배송비만 빼서 환불
        2. 배송비 동봉
    3. 금액, 주소지등 확인 필요
11. 환불 상태변경 → 6에서 생성한 api활용 or 환불용 api 생성
    1. 
12. 환불취소
13. 교환등록
    1. 등록시 바로 교환 회수요청으로 진행
    2. 교환 내용
        1. 사이즈 변경
        2. 동일 사이즈에서 제품만 변경
    3. 금액, 주소지등 확인 필요
14. 교환취소


---

## 구현예정 기능 리스트

구현 우선순위

로그인 -> 상품 → 상품조회 → 주문 → 장바구니 → 상품(심화) → 회원 → 로그인(심화) → 그외

1. 상품
    1. 상품 카테고리
        1. 1뎁스
            1. 남성, 여성, 인테리어 등등
        2. 2뎁스
            1. 의류, 가방, 악세서리 등
        3. 3뎁스
            1. 상하의, 아우터 등
        4. 4뎁스
            1. 맨투맨, 니트 등..
    2. 상품등록 (브랜드)
        1. 기본정보
            1. 상품정보
            2. 태그
            3. 사이즈
            4. 재고
    3. 상품수정 (브랜드)
    4. 상품삭제 (브랜드)
    5. 상품 목록 조회
        1. 특정 카테고리 아이템들 추천순으로 나열
        2. 필터
            1. 가격대
            2. 색상
            3. 브랜드
            4. 이름
    6. 상품 상세
        1. 별점, 후기, 좋아요
        2. 태그 → 검색용?
2. 주문
    1. 주문 등록
    2. 주문 목록 기간별로 조회
    3. 주문 취소
    4. 주문 상태(일반주문, 교환, 반품(환불))
    5. 주문상태로그
    6. 쿠폰
    7. 적립금
4. 장바구니
5. 회원
    1. 일반회원
        1. 회원 가입, 탈퇴
        2. 회원권한
            1. 구매자
            2. 비회원
        3. 회원등급
            1. 등급별 결제시 할인
        4. 회원 정보
            1. 주문내역
            2. 회원 정보
            3. 취소/교환/반품
            4. 리뷰
            5. 마일리지
            6. 좋아요
    2. 브랜드
        1. 브랜드 가입
        2. 브랜드 탈퇴
        3. 브랜드 직원 가입
        4. 브랜드 직원 권한
            1. 관리자
            2. 일반사원
        5. 브랜드 직원에서 탈퇴
            1. 관리자가 삭제?
            2. 일반 사원 계정으로 자진삭제 가능?
        6. 브랜드 정보
6. 로그인
    1. 로그인
    2. 로그아웃


---
## Use Case


