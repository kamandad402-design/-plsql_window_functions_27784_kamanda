## sql query 
CREATE DATABASE student_performance;


CREATE TABLE students (
    student_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name  VARCHAR(50) NOT NULL,
    enrollment_date DATE NOT NULL,
    department VARCHAR(100) NOT NULL
);


CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_code VARCHAR(20) NOT NULL,
    course_name VARCHAR(100) NOT NULL,
    instructor VARCHAR(100) NOT NULL,
    academic_term VARCHAR(50) NOT NULL
);


CREATE TABLE student_grades (
    grade_id INT PRIMARY KEY,
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    assessment_date DATE NOT NULL,
    assessment_type VARCHAR(50) NOT NULL,
    score NUMERIC(5,2) NOT NULL,
    max_score NUMERIC(5,2) NOT NULL,

    CONSTRAINT fk_student
        FOREIGN KEY (student_id)
        REFERENCES students(student_id),

    CONSTRAINT fk_course
        FOREIGN KEY (course_id)
        REFERENCES courses(course_id)
);



CREATE INDEX idx_student_grades_student
ON student_grades(student_id);

CREATE INDEX idx_student_grades_course
ON student_grades(course_id);

CREATE INDEX idx_student_grades_date
ON student_grades(assessment_date);


INSERT INTO students VALUES
(1001, 'Alice', 'Keza', '2024-01-15', 'Computer Science'),
(1002, 'Kiza', 'Habana', '2024-01-15', 'Computer Science'),
(1003, 'Claire', 'Uwase', '2024-01-15', 'Information Systems');



INSERT INTO courses VALUES
(2001, 'INSY8311', 'Database Development', 'Eric Maniraguha', 'Fall 2024'),
(2002, 'INSY8312', 'Advanced PL/SQL', 'Eric Maniraguha', 'Fall 2024'),
(2003, 'INSY8313', 'Data Warehousing', 'Eric Maniraguha', 'Fall 2024');
INSERT INTO student_grades VALUES
-- Student 1001
(3001, 1001, 2001, '2024-09-10', 'Quiz 1', 85.5, 100),
(3002, 1001, 2001, '2024-09-20', 'Midterm Exam', 92.0, 100),
(3003, 1001, 2001, '2024-10-15', 'Final Exam', 88.5, 100),

-- Student 1002
(3004, 1002, 2001, '2024-09-10', 'Quiz 1', 78.0, 100),
(3005, 1002, 2001, '2024-09-20', 'Midterm Exam', 82.0, 100),
(3006, 1002, 2001, '2024-10-15', 'Final Exam', 85.0, 100),

-- Student 1003
(3007, 1003, 2001, '2024-09-10', 'Quiz 1', 92.5, 100),
(3008, 1003, 2001, '2024-09-20', 'Midterm Exam', 95.0, 100),
(3009, 1003, 2001, '2024-10-15', 'Final Exam', 94.0, 100);

PART A — SQL JOIN QUERIES (pgAdmin)
### RIGHT JOIN======================================================
SELECT 
    s.student_id,
    s.first_name,
    s.last_name,
    c.course_name,
    g.assessment_type,
    g.score
FROM student_grades g
INNER JOIN students s ON g.student_id = s.student_id
INNER JOIN courses c ON g.course_id = c.course_id;

### FULL OUTER JOIN===============================================

SELECT 
    s.student_id,
    s.first_name,
    s.last_name
FROM students s
LEFT JOIN student_grades g
    ON s.student_id = g.student_id
WHERE g.grade_id IS NULL;

### SELF JOIN================================================

SELECT 
    s.student_id,
    s.first_name,
    g.grade_id,
    g.assessment_type
FROM students s
FULL OUTER JOIN student_grades g
    ON s.student_id = g.student_id;
### Ranking Functions====================================================

SELECT 
    s1.first_name AS student_one,
    s2.first_name AS student_two,
    s1.department
FROM students s1
JOIN students s2
    ON s1.department = s2.department

   AND s1.student_id <> s2.student_id;
### Aggregate Window Functions========================================
SELECT
    s.student_id,
    s.first_name,
    g.score,
    RANK() OVER (ORDER BY g.score DESC) AS rank_position
FROM student_grades g
JOIN students s ON g.student_id = s.student_id
WHERE g.assessment_type = 'Final Exam';

### Navigation Functions=============================================
SELECT
    s.student_id,
    s.first_name,
    g.assessment_date,
    g.score,
    AVG(g.score) OVER (
        PARTITION BY g.student_id
        ORDER BY g.assessment_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_average
FROM student_grades g
JOIN students s ON g.student_id = s.student_id;

### Distribution Functions====================================

SELECT
    student_id,
    total_score,
    NTILE(4) OVER (ORDER BY total_score DESC) AS performance_quartile
FROM (
    SELECT
        s.student_id,
        SUM(g.score) AS total_score
    FROM students s
    JOIN student_grades g ON s.student_id = g.student_id
    GROUP BY s.student_id
) sub;
### Moving Average========================================
SELECT
    s.student_id,
    s.first_name,
    g.assessment_date,
    g.score,
    AVG(g.score) OVER (
        PARTITION BY g.student_id
        ORDER BY g.assessment_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_average
FROM student_grades g
JOIN students s ON g.student_id = s.student_id;
