<h3>
    1. 테이블 생성
</h3>



1) user

**테이블 생성**

~~~sql

CREATE TABLE USER
(
	UserId               VARCHAR2(20)  		   	NOT NULL,
	UserName             VARCHAR2(10)			NOT NULL,
	Age                  NUMBER					NULL,
	Region               VARCHAR2(10)			NULL,
	Phone                VARCHAR2(20)			NULL,
	CONSTRAINTS PK_USER PRIMARY KEY(UserId)
);
~~~

2) Product

**테이블생성**

~~~sql
CREATE TABLE PRODUCT
(
	ProductId            NUMBER              	NOT NULL,
	ProductName          VARCHAR2(10)			NOT NULL,
	Price                NUMBER					NULL,
	CONSTRAINTS PK_Product PRIMARY KEY(ProductId)
);

~~~

3) Buy

**테이블생성**

~~~sql
CREATE TABLE PURCHASE
(
	PurchaseId           NUMBER             	NOT NULL,
    UserId               VARCHAR2(20) 		   	NOT NULL,
    ProductId            NUMBER              	NOT NULL,
    TimeStamp            CHAR(15)				NOT NULL,
	CONSTRAINTS PK_Purchase PRIMARY KEY(PurchaseId)
);

~~~

<h3>
 2. 고객별 구매금액 총액 조회
</h3>



```sql
SELECT U.UserId, sum(P.Price) Total
FROM EDU_USER U, EDU_PRODUCT P, EDU_PURCHASE C
WHERE U.UserId = C.UserId
AND P.ProductId = C.ProductId
GROUP BY U.UserId;

--RevId 추가 후

SELECT U.UserId, sum(P.Price) Total
FROM EDU_USER U, EDU_PRODUCT P, EDU_PURCHASE C
WHERE U.UserId = C.UserId
AND U.RevId = 0
AND P.ProductId = C.ProductId
AND P.RevId = 0
GROUP BY U.UserId;
```



<h3>
    3. 구매가능 아이템 추가
</h3>



~~~sql
INSERT INTO PRODUCT(ProductId,ProductName,Price) VALUES(5,'형광펜',750);

--RevId 추가 후
INSERT INTO PRODUCT(ProductId,RevId,ProductName,Price,MaxRevId) VALUES(5,0,'형광펜',750,1);
INSERT INTO PRODUCT(ProductId,RevId,ProductName,Price,MaxRevId) VALUES(5,1,'형광펜',750,1);

~~~



<h3>
    4. 해가 지나 나이를 1살씩 증가
</h3>



~~~sql
UPDATE USER
SET Age = Age+1;

--RevId 추가 후
UPDATE USER
SET Age = Age+1
AND MaxRevId = MaxRevId+1
WHERE RevId = 0

INSERT INTO USER ( UserId, RevId, UserName, Age, Region, Phone, MaxRevId)
 SELECT UserId, MaxRevId, UserName, Age, Region, Phone, MaxRevId
 FROM USER
 WHERE RevId = 0;
~~~



<h3>
    5. 나이 대신 출생연도를 관리
</h3>



~~~sql
ALTER TABLE EDU_USER MODIFY( Age NUMBER(10));

UPDATE EDU_USER
SET AGE = (SELECT EXTRACT(YEAR FROM SYSDATE)
           FROM dual) - Age + 1
~~~





<h3>
    6. 가격이 결정되지 않은 신규물품 추가할 경우, </br>
    테이블 구조 OR 데이터 어떻게 변경
</h3>



~~~sql
ALTER TABLE EDU_PRODUCT MODIFY( PRICE NULL);

~~~



<h3>
    7. 지우개가 800원에서 1.25달러로 변경  
</h3>


~~~sql
ALTER TABLE EDU_PRODUCT ADD CURRENCY CHAR(4) NULL;
ALTER TABLE EDU_PRODUCT MODIFY (PRICE NUMBER);

UPDATE EDU_PRODUCT
SET CURRENCY = 'KRW'

UPDATE EDU_PRODUCT 
SET PRICE = 1.25
, CURRENY = 'USD'
WHERE PRODUCTNAME = '지우개';


~~~

<h3>
    8. ITS 이력관리 기준에 맞게 테이블 변경  
</h3>
1) user

~~~sql
CREATE TABLE USER
(
	UserId               VARCHAR2(10)   	   	NOT NULL,
    RevId				 NUMBER(4)				NOT NULL,
	UserName             VARCHAR2(10)			NOT NULL,
	Age                  NUMBER(10)				NULL,
	Region               VARCHAR2(10)			NULL,
	Phone                VARCHAR2(20)			NULL,
    MaxRevId			 NUMBER(4)				NOT NULL,
	CONSTRAINTS PK_USER PRIMARY KEY(UserId,RevId)
);
~~~

2) Product

~~~sql
CREATE TABLE PRODUCT
(
	ProductId            NUMBER(8)             	NOT NULL,
	RevId				 NUMBER(4)				NOT NULL,
    ProductName          VARCHAR2(10)			NOT NULL,
	Price                NUMBER(10)				NULL,
    MaxRevId			 NUMBER(4)				NOT NULL,
	CONSTRAINTS PK_Product PRIMARY KEY(ProductId,RevId)
);

~~~

3) Buy

~~~sql
CREATE TABLE PURCHASE
(
	PurchaseId           NUMBER(10)            	NOT NULL,
    RevId				 NUMBER(4)				NOT NULL,
    UserId               VARCHAR2(10)  		   	NOT NULL,
    ProductId            NUMBER(8)             	NOT NULL,
    TimeStamp            CHAR(15)				NOT NULL,
    MaxRevId			 NUMBER(4)				NOT NULL,
	CONSTRAINTS PK_Purchase PRIMARY KEY(PurchaseId,RevId)
);

~~~
