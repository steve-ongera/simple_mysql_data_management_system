# MySQL Student Management System - Command Guide

## Database Setup
```sql
-- Create database
CREATE DATABASE student_management;
USE student_management;

-- Create students table
CREATE TABLE students (
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    date_of_birth DATE,
    email VARCHAR(100) UNIQUE,
    registration_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create courses table
CREATE TABLE courses (
    course_id INT PRIMARY KEY AUTO_INCREMENT,
    course_name VARCHAR(100) NOT NULL,
    course_code VARCHAR(20) UNIQUE,
    credits INT
);

-- Create enrollments table
CREATE TABLE enrollments (
    enrollment_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

## Basic Commands

### Insert Data
```sql
-- Add a student
INSERT INTO students (first_name, last_name, email) 
VALUES ('John', 'Doe', 'john.doe@email.com');

-- Add a course
INSERT INTO courses (course_name, course_code, credits) 
VALUES ('Introduction to Programming', 'CS101', 3);

-- Enroll student in course
INSERT INTO enrollments (student_id, course_id, enrollment_date) 
VALUES (1, 1, CURRENT_DATE());
```

### Select Data
```sql
-- View all students
SELECT * FROM students;

-- View specific student
SELECT * FROM students WHERE student_id = 1;

-- View students in a specific course
SELECT s.first_name, s.last_name, c.course_name
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id
WHERE c.course_code = 'CS101';
```

## Intermediate Commands

### Update Records
```sql
-- Update student email
UPDATE students 
SET email = 'new.email@email.com' 
WHERE student_id = 1;

-- Update course credits
UPDATE courses 
SET credits = 4 
WHERE course_code = 'CS101';
```

### Delete Records
```sql
-- Remove enrollment
DELETE FROM enrollments 
WHERE student_id = 1 AND course_id = 1;

-- Delete student (requires removing enrollments first)
DELETE FROM enrollments WHERE student_id = 1;
DELETE FROM students WHERE student_id = 1;
```

## Advanced Commands

### Complex Queries
```sql
-- Count students per course
SELECT c.course_name, COUNT(e.student_id) as student_count
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.course_id;

-- Find students enrolled in multiple courses
SELECT s.student_id, s.first_name, s.last_name, COUNT(e.course_id) as course_count
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id
HAVING course_count > 1;
```

### Views
```sql
-- Create view for student enrollment details
CREATE VIEW student_enrollment_details AS
SELECT 
    s.student_id,
    s.first_name,
    s.last_name,
    c.course_name,
    c.course_code,
    e.enrollment_date
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id;
```

### Stored Procedures
```sql
-- Create procedure to enroll student
DELIMITER //
CREATE PROCEDURE enroll_student(
    IN p_student_id INT,
    IN p_course_code VARCHAR(20)
)
BEGIN
    DECLARE v_course_id INT;
    
    -- Get course ID
    SELECT course_id INTO v_course_id 
    FROM courses 
    WHERE course_code = p_course_code;
    
    -- Insert enrollment
    INSERT INTO enrollments (student_id, course_id, enrollment_date)
    VALUES (p_student_id, v_course_id, CURRENT_DATE());
END //
DELIMITER ;
```

### Indexes
```sql
-- Create index for faster email searches
CREATE INDEX idx_student_email ON students(email);

-- Create index for course lookup
CREATE INDEX idx_course_code ON courses(course_code);
```

### Backup and Restore
```bash
# Backup database
mysqldump -u username -p student_management > backup.sql

# Restore database
mysql -u username -p student_management < backup.sql
```