## スキーマの作成
CREATE SCHEMA school;

## テーブルの作成
CREATE TABLE school.class_1 (
	class_num integer NOT NULL DEFAULT 1,
	student_num serial NOT NULL,
	student_name varchar(20) NOT NULL,
	gender_type varchar(6) NOT NULL,
	score_janapese integer NOT NULL,
	score_math integer NOT NULL,
	score_society integer NOT NULL,
	score_science integer NOT NULL);

## 性別テーブルの作成
CREATE TABLE school.gender (gender_type varchar(1) NOT NULL,
					  gender_name varchar(6) NOT NULL,
					 CONSTRAINT pk_school_gender PRIMARY KEY (gender_type));


## 性別テー物にINSERT
INSERT INTO school.gender (gender_type,gender_name) VALUES
	('M','MALE'),
	('F','FEMALE'),
	('O','OTHER');

## テーブルの型を変更
ALTER TABLE school.class_1 ALTER COLUMN gender_type SET DATA TYPE varchar(1);

## テーブルにFKをつける
ALTER TABLE school.class_1
    ADD CONSTRAINT fk_class_1_gender FOREIGN KEY (gender_type)
    REFERENCES school.gender
    ON UPDATE NO ACTION
    ON DELETE NO ACTION;

## 性別テーブルのSELECT
SELECT 
	gender.gender_type,
	gender.gender_name
FROM
	school.gender;

ALTER TABLE school.class_1
    RENAME score_janapese to score_japanese;

## テーブルの全件表示
SELECT 
	class_1.class_num,
	class_1.student_num,
	class_1.student_name,
	class_1.gender_type,
	class_1.score_japanese,
	class_1.score_math,
	class_1.score_society,
	class_1.score_science
FROM 
	school.class_1

## テーブルの平均を求める
SELECT 
	AVG(class_1.score_japanese),
	AVG(class_1.score_math),
	AVG(class_1.score_society),
	AVG(class_1.score_science)
	
FROM 
	school.class_1;


SELECT 
	class_1.gender_type, 
	MAX(class_1.score_japanese),
	MAX(class_1.score_math),
	MAX(class_1.score_science),
	MAX(class_1.score_society)
FROM 
	school.class_1
GROUP BY	
	class_1.gender_type;


SELECT
	class_1.gender_type,
	AVG(class_1.score_japanese) AS japanese,
	AVG(class_1.score_math) AS math,
	AVG(class_1.score_science) AS science,
	AVG(class_1.score_society) AS society
FROM 
	school.class_1
GROUP BY
	class_1.gender_type;


SELECT
	class_1.gender_type,
	AVG(class_1.score_japanese)
FROM 
	school.class_1
WHERE 
	class_1.score_japanese <= 50
GROUP BY
	class_1.gender_type;


SELECT
	class_1.gender_type,
	MAX(class_1.score_japanese)
FROM 
	school.class_1
WHERE 
	class_1.score_japanese <= 50
GROUP BY
	class_1.gender_type;



SELECT 
	class_1.student_num,
	class_1.student_name,
	
MAX(class_1.score_japanese) AS japanese

FROM school.class_1

GROUP BY 
	class_1.student_name,
	class_1.student_num

HAVING MAX(class_1.score_japanese) <=50

ORDER BY
	class_1.student_num;

## トランザクションその１
    「新田新一朗と佐藤萌の国語の点数が入れ替わっていた」という設定で点数の修正をする。
BEGIN;
UPDATE school.class_1 SET score_japanese = 70 WHERE student_num = 1;
COMMIT;
BEGIN;
UPDATE school.class_1 SET score_japanese = 60 WHERE student_num = 2;
COMMIT;

カラムupdate_atの追加	
ALTER TABLE school.class_1 ADD update_at timestamp without time zone DEFAULT now();


## トリガー関数の作成
CREATE FUNCTION school.update_at() 
RETURNS trigger 
LANGUAGE 'plpgsql'
AS $BODY$
BEGIN
	NEW.update_at = current_timestamp;
	RETURN NEW;
END 
$BODY$;

CREATE TRIGGER school_update_at BEFORE UPDATE
ON school.class_1
FOR EACH ROW
EXECUTE FUNCTION school.update_at()


## トランザクションその２
BEGIN;
UPDATE school.class_1 SET score_japanese = 70 WHERE student_num = 5;
UPDATE school.class_1 SET score_japanese = 60 WHERE student_num = 6;
COMMIT;

## トランザクションその３
BEGIN;
UPDATE school.class_1 SET score_society = 70 WHERE student_num = 7;
ROLLBACK;
BEGIN;
UPDATE school.class_1 SET score_society = 70 WHERE student_num = 8;
COMMIT;8

## トランザクションその４
BEGIN;
UPDATE school.class_1 SET score_society = 70 WHERE student_num = 9;
SAVEPOINT correction;
	UPDATE school.class_1 SET score_society = 70 WHERE student_num = 10;
	ROLLBACK TO SAVEPOINT correction;
UPDATE school.class_1 SET score_society = 60 WHERE student_num = 10;
COMMIT;

## トランザクションその５
BEGIN ISOLATION LEVEL SERIALIZABLE READ WRITE;
UPDATE school.class_1 SET score_society = 70 WHERE student_num = 9;
SAVEPOINT correction;
	UPDATE school.class_1 SET score_society = 70 WHERE student_num = 10;
	ROLLBACK TO SAVEPOINT correction;
UPDATE school.class_1 SET score_society = 60 WHERE student_num = 10;
COMMIT;

## NO ACTION
ALTER TABLE practice.owners  ADD CONSTRAINT fk_owner_gender 
	FOREIGN KEY (owner_gender_type) 
	REFERENCES practice.gender 
	ON UPDATE NO ACTION
	ON DELETE NO ACTION;

UPDATE practice.gender SET gender_type = 'Frau' WHERE gender_type = 'FEMALE';
    ERROR:  テーブル"gender"の更新または削除は、テーブル"owners"の外部キー制約"fk_owner_gender"に違反します
DETAIL:  キー(gender_type)=(FEMALE)はまだテーブル"owners"から参照されています
SQL 状態：23503

DELETE FROM practice.gender WHERE gender_type = 'MALE';
    ERROR:  テーブル"gender"の更新または削除は、テーブル"owners"の外部キー制約"fk_owner_gender"に違反します
    DETAIL:  キー(gender_type)=(MALE)はまだテーブル"owners"から参照されています
    SQL 状態：23503

## RESTRICT
ALTER TABLE practice.owners  ADD CONSTRAINT fk_owner_gender 
	FOREIGN KEY (owner_gender_type) 
	REFERENCES practice.gender 
	ON UPDATE RESTRICT
	ON DELETE RESTRICT;

DELETE FROM practice.gender WHERE gender_type = 'MALE';
    ERROR:  テーブル"gender"の更新または削除は、テーブル"owners"の外部キー制約"fk_owner_gender"に違反します
    DETAIL:  キー(gender_type)=(MALE)はまだテーブル"owners"から参照されています
    SQL 状態：23503

UPDATE practice.gender SET gender_type = 'Frau' WHERE gender_type = 'FEMALE';
    ERROR:  テーブル"gender"の更新または削除は、テーブル"owners"の外部キー制約"fk_owner_gender"に違反します
    DETAIL:  キー(gender_type)=(FEMALE)はまだテーブル"owners"から参照されています
    SQL 状態：23503


## CASCADE
ALTER TABLE practice.owners  ADD CONSTRAINT fk_owner_gender 
	FOREIGN KEY (owner_gender_type) 
	REFERENCES practice.gender 
	ON UPDATE CASCADE
	ON DELETE CASCADE;

UPDATE practice.gender SET gender_type = 'Frau' WHERE gender_type = 'FEMALE';

    UPDATE 1

    クエリが 53 msec で成功しました

## SET NULL
ALTER TABLE practice.owners  ADD CONSTRAINT fk_owner_gender 
	FOREIGN KEY (owner_gender_type) 
	REFERENCES practice.gender 
	ON UPDATE CASCADE
	ON DELETE SET NULL;

SELECT 
	employee2.first_name,
	employee2.last_name,
	employee2.department_id
FROM 
	employee.employee2
INTERSECT

SELECT
	employee2.first_name,
	employee2.last_name,
	employee2.department_id
FROM 
	employee.employee2

SELECT 
	employee2.first_name,
	employee2.last_name,
	employee2.department_id
FROM 
	employee.employee2
UNION

SELECT
	employee2.first_name,
	employee2.last_name,
	employee2.department_id
FROM 
	employee.employee2

## employee1作り
UPDATE employee.employee1 SET (first_name,last_name,date_of_birth,department_id,gender_type) = ('フローレス','ナイチンゲール','1820-05-12',1002,'FEMALE') WHERE employee_id = 1;
