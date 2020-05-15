20200515-SQL
/*employee1 × employee2
UNION*/

SELECT 
	employee2.employee_id,
	employee2.last_name,
	employee2.first_name,
	employee2.date_of_birth,
	employee2.gender_type,
	employee2.department_id,
	employee2.updated_at
FROM
	employee.employee2
UNION[ALL]
SELECT 
	employee1.employee_id,
	employee1.last_name,
	employee1.first_name,
	employee1.date_of_birth,
	employee1.gender_type,
	employee1.department_id,
	employee1.updated_at
FROM 
	employee.employee1
ORDER BY
	employee_id

/*INTERSECT*/
SELECT 
	employee2.employee_id,
	employee2.last_name,
	employee2.first_name,
	employee2.date_of_birth,
	employee2.gender_type,
	employee2.department_id,
	employee2.updated_at
FROM
	employee.employee2
INTERSECT
SELECT 
	employee1.employee_id,
	employee1.last_name,
	employee1.first_name,
	employee1.date_of_birth,
	employee1.gender_type,
	employee1.department_id,
	employee1.updated_at
FROM 
	employee.employee1
ORDER BY
	employee_id

/*EXCEPT*/

SELECT 
	employee2.employee_id,
	employee2.last_name,
	employee2.first_name,
	employee2.date_of_birth,
	employee2.gender_type,
	employee2.department_id,
	employee2.updated_at
FROM
	employee.employee2
EXCEPT
SELECT 
	employee1.employee_id,
	employee1.last_name,
	employee1.first_name,
	employee1.date_of_birth,
	employee1.gender_type,
	employee1.department_id,
	employee1.updated_at
FROM 
	employee.employee1
ORDER BY
	employee_id

/*employee2 × class_1
UNION*/

SELECT 
	employee2.employee_id,
	employee2.first_name,
	employee2.gender_type,
	employee2.updated_at
FROM
	employee.employee2
UNION
 ALL
SELECT 
	class_1.student_num,
	class_1.student_name,
	class_1.gender_type,
	class_1.update_at
FROM 
	school.class_1
ORDER BY
	employee_id,
	student_num

/*INTERSECT*/
SELECT 
	employee2.employee_id,
	employee2.first_name,
	employee2.gender_type,
	employee2.updated_at
FROM
	employee.employee2
INTERSECT
SELECT 
	class_1.student_num,
	class_1.student_name,
	class_1.gender_type,
	class_1.update_at
FROM 
	school.class_1
ORDER BY

/* EXCEPT*/
SELECT 
	employee2.employee_id,
	employee2.first_name,
	employee2.gender_type,
	employee2.updated_at
FROM
	employee.employee2
EXCEPT
SELECT 
	class_1.student_num,
	class_1.student_name,
	class_1.gender_type,
	class_1.update_at
FROM 
	school.class_1
ORDER BY
	employee_id

/*LIMITとOFFSET*/
SELECT
	employee2.employee_id,
	employee2.last_name,
	employee2.first_name,
	employee2.date_of_birth,
	employee2.gender_type,
	employee2.department_id,
	employee2.updated_at
FROM
	employee.employee2
ORDER BY
	employee_id
LIMIT 3 OFFSET 2;


/*UNION + LIMIIT OFFSET*/
SELECT
	employee2.employee_id,
	employee2.last_name,
	employee2.first_name,
	employee2.date_of_birth,
	employee2.gender_type,
	employee2.department_id,
	employee2.updated_at
FROM
	employee.employee2
UNION
SELECT 
	employee1.employee_id,
	employee1.last_name,
	employee1.first_name,
	employee1.date_of_birth,
	employee1.gender_type,
	employee1.department_id,
	employee1.updated_at
FROM 
	employee.employee1
ORDER BY
	employee_id
LIMIT 5 OFFSET 5

/*サブクエリ
WHERE句*/
SELECT 
	employee1.employee_id,
	employee1.last_name,
	employee1.first_name,
	employee1.date_of_birth,
	employee1.gender_type,
	employee1.department_id,
	employee1.updated_at
FROM 
	employee.employee1
WHERE department_id =
         (SELECT department_id FROM employee.department
          WHERE department_name = '営業部');

/*SELECT後のサブクエリ*/
SELECT 
	class_1.class_num,
	class_1.student_num,
	class_1.student_name,
	class_1.gender_type,
	class_1.score_japanese,
	class_1.score_math,
	class_1.score_society,
	class_1.score_science,
	 (SELECT 
	  	AVG(class_1.score_japanese) AS japanese
	  FROM school.class_1),
	  (SELECT 
	  	AVG(class_1.score_math) AS math
	  FROM school.class_1),
	  (SELECT 
	  	AVG(class_1.score_society) AS society
	  FROM school.class_1),
	  (SELECT 
	  	AVG(class_1.score_science) AS science
	  FROM school.class_1)
FROM 
	school.class_1

/*FROM後のサブクエリ*/
SELECT student_name

FROM (SELECT * FROM school.class_1 WHERE score_math > 50) AS japanese ;

/*相関サブクエリ*/
SELECT * FROM employee.employee1 AA
        WHERE  IN
        (SELECT department_id
        FROM employee.employee2 BB
        WHERE AA.employee_id = BB.employee_id);

/*年齢の自動入力 失敗*/
CREATE FUNCTION employee.years()
    RETURNS trigger
    LANGUAGE 'plpgsql'
    VOLATILE NOT LEAKPROOF
AS $BODY$
BEGIN
	NEW.years = age('timestamp date_of_birth');
	RETURN NEW;
END$BODY$;

CREATE TRIGGER employee_years
    AFTER INSERT
    ON employee.employee1
    FOR EACH ROW
    EXECUTE PROCEDURE employee.years();

ALTER TABLE employee.employee1 ADD COLUMN years date;

INSERT INTO employee.employee1 (first_name,last_name,date_of_birth,department_id,gender_type)VALUES
	('萌','佐藤','1997-05-13',1002,'FEMALE');

    ERROR:  関数 age(unknown) は一意ではありません
    LINE 1: SELECT age('timestamp date_of_birth')
                ^
    HINT:  最善の候補関数を選択できませんでした。明示的な型キャストが必要かもしれません
    QUERY:  SELECT age('timestamp date_of_birth')
    CONTEXT:  PL/pgSQL関数employee.years()の3行目 - 代入
    SQL 状態：42725

SELECT 
	employee1.employee_id,
	employee1.last_name,
	employee1.first_name,
	employee1.date_of_birth,
	employee1.gender_type,
	employee1.department_id,
	employee1.updated_at,
	(SELECT age('timestamp date_of_birth'))
FROM employee.employee1

    ERROR:  関数 age(unknown) は一意ではありません
    LINE 9:  (SELECT age('timestamp date_of_birth')
                    ^
    HINT:  最善の候補関数を選択できませんでした。明示的な型キャストが必要かもしれません
    SQL 状態：42725
    文字：186


/*LIKEをつかったあいまい検索をサーブレットで　成功*/
if(submit.equals("検索")){
            List<BooksBeans> list = new ArrayList<BooksBeans>();
            final String sql="SELECT * FROM library.books where bookname LIKE ? ";

            try(Connection conn = dataSource.getConnection();
                PreparedStatement statement = conn.prepareStatement(sql);
                ){
                String serchBookName = request.getParameter("bookName");
                statement.setString(1,serchBookName + '%');
                ResultSet result = statement.executeQuery();