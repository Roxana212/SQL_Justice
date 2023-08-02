# SQL_Justice
-- Create the judge table
CREATE TABLE judges (
  judge_id INT PRIMARY KEY,
  judge_name VARCHAR(100),
  years_of_experience INT,
  court_id INT,
  FOREIGN KEY (court_id) REFERENCES courts(court_id)
);

INSERT INTO judges (judge_id, judge_name, years_of_experience, court_id)
VALUES
  (1, 'Judge Popescu', 10, 1),
  (2, 'Judge Dumitru', 8, 2),
  (3, 'Judge Ilie', 12, 3),
  (4, 'Judge Toma', 6, 4);

--Create the courts table
CREATE TABLE courts (
  court_id INT PRIMARY KEY,
  court_name VARCHAR(100),
  location VARCHAR(100)
);

INSERT INTO courts (court_id, court_name, location)
VALUES
  (1, 'Court of Appeal', 'Bucharest'),
  (2, 'Court of Appeal', 'Cluj-Napoca'),
  (3, 'High Court of Cassation and Justice', 'Bucharest'),
  (4, 'Court of Appeal', 'Timisoara');

-- Create the lawyers table
CREATE TABLE lawyers (
  lawyer_id INT PRIMARY KEY,
  lawyer_name VARCHAR(100),
  specialization VARCHAR(100),
  years_of_practice INT
);

INSERT INTO lawyers (lawyer_id, lawyer_name, specialization, years_of_practice)
VALUES
  (1, 'Ana Pop', 'Criminal Law', 8),
  (2, 'Marian Marin', 'Civil Law', 10),
  (3, 'Andreea Mihai', 'Family Law', 6),
  (4, 'Andrei Ion', 'Labor Law', 12);

--Create the judicial cases table
CREATE TABLE judicial_cases (
  case_id INT PRIMARY KEY,
  case_name VARCHAR(200),
  opening_date DATE,
  closing_date DATE,
  court_id INT,
  judge_id INT,
  lawyer_id INT,
  client_id INT,
  FOREIGN KEY (court_id) REFERENCES courts(court_id),
  FOREIGN KEY (judge_id) REFERENCES judges(judge_id),
  FOREIGN KEY (lawyer_id) REFERENCES lawyers(lawyer_id),
  FOREIGN KEY (client_id) REFERENCES clients(client_id)
);

INSERT INTO judicial_cases (case_id, case_name, opening_date, closing_date, court_id, judge_id, lawyer_id, client_id)
VALUES
  (1, 'Case 1', '2023-01-10', '2023-02-15', 1, 1, 1, 1),
  (2, 'Case 2', '2023-02-20', NULL, 2, 2, 2, 2),
  (3, 'Case 3', '2023-03-05', '2023-04-20', 3, 3, 3, 3),
  (4, 'Case 4', '2023-04-25', NULL, 4, 4, 4, 4),
  (5, 'Case 5', '2023-05-12', '2023-07-18', 1, 1, 2, 2),
  (6, 'Case 6', '2023-06-03', NULL, 2, 2, 3, 1),
  (7, 'Case 7', '2023-07-01', NULL, 3, 3, 4, 3);

--Create clients table
CREATE TABLE clients (
  client_id INT PRIMARY KEY,
  client_name VARCHAR(100),
  lawyer_id INT,
  case_fee INT,
  FOREIGN KEY (lawyer_id) REFERENCES lawyers(lawyer_id)
);

INSERT INTO clients (client_id, client_name, lawyer_id, case_fee)
VALUES
  (1, 'Client A', 1, 10000),
  (2, 'Client B', 2, 15000),
  (3, 'Client C', 3, 20000),
  (4, 'Client D', 4, 25000),
  (5, 'Client E', 4, 20000),
  (6, 'Client F', 4, 30000),
  (7, 'Client G', 3, 10000),
  (8, 'Client H', 3, 20000),
  (9, 'Client I', 1, 30000);

--Cases that are still open
SELECT *
FROM judicial_cases
WHERE closing_date IS NULL;

--The most recent 10 cases 
SELECT case_id, case_name, opening_date, closing_date
FROM judicial_cases 
ORDER BY opening_date DESC
LIMIT 10;

--All the judicial cases along with the names of the judges handling them
SELECT jc.case_id, jc.case_name, jc.opening_date, jc.closing_date, j.judge_name
FROM judicial_cases AS jc
JOIN judges AS j ON jc.judge_id = j.judge_id;

--Number of cases handled by each judge
SELECT jc.judge_id, j.judge_name, COUNT(*) AS total_cases
FROM judicial_cases AS jc
JOIN judge AS j ON jc.judge_id = j.judge_id
GROUP BY jc.judge_id, j.judge_name;

-- The lawyers and the number of cases they are representing
SELECT jc.lawyer_id, l.lawyer_name, COUNT(*) AS total_cases_represented
FROM judicial_cases AS jc
JOIN lawyer AS l ON jc.lawyer_id=l.lawyer_id
GROUP BY jc.lawyer_id, l.lawyer_name;

--Number of cases filed in each court along with the number of cases closed in that court
SELECT c.court_id, c.court_name, COUNT(jc.case_id) AS total_cases_filed, 
COUNT(CASE 
	WHEN jc.closing_date IS NOT NULL THEN jc.case_id 
	END) AS total_cases_closed
FROM courts AS c
LEFT JOIN judicial_cases AS jc ON c.court_id = jc.court_id
GROUP BY c.court_id, c.court_name;

--The top 3 lawyers with the most closed cases, along with the number of cases they represented and the total fees they earned
SELECT l.lawyer_id, l.lawyer_name, COUNT(jc.case_id) AS total_cases_represented, 
SUM(c.case_fee) AS total_fees_earned
FROM lawyers AS l
LEFT JOIN judicial_cases AS jc ON l.lawyer_id = jc.lawyer_id
LEFT JOIN clients AS c ON jc.client_id = c.client_id
WHERE jc.closing_date IS NOT NULL
GROUP BY l.lawyer_id, l.lawyer_name
ORDER BY total_cases_represented DESC
LIMIT 3;

--The top 3 courts with the most open cases 
SELECT c.court_name, COUNT(jc.case_id) AS total_open_cases
FROM courts AS c
LEFT JOIN judicial_cases AS jc ON c.court_id = jc.court_id
WHERE jc.closing_date IS NULL
GROUP BY c.court_name
ORDER BY total_open_cases DESC
LIMIT 3;
