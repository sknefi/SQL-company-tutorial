CREATE TABLE employee(
    emp_id INT,
    first_name VARCHAR(25),
    last_name VARCHAR(25),
    birthdate DATE,
    sex VARCHARACTER(1),
    salary INT,
    superV_id INT,
    branch_id INT,

    PRIMARY KEY(emp_id)
);

CREATE TABLE branch(
    branch_id INT,
    branch_name VARCHAR(30),
    manager_id INT,
    manager_start_date DATE,

    PRIMARY KEY(branch_id),
    FOREIGN KEY(manager_id) REFERENCES employee(emp_id) ON DELETE SET NULL
);


ALTER TABLE employee
ADD FOREIGN KEY(branch_id) REFERENCES branch(branch_id)
ON DELETE SET NULL;


ALTER TABLE employee
ADD FOREIGN KEY(superV_id) REFERENCES employee(emp_id)
ON DELETE SET NULL;

ALTER TABLE branch
MODIFY manager_start_date DATE;


CREATE TABLE client(
    client_id INT,
    client_name VARCHAR(25),
    branch_id INT,

    PRIMARY KEY(client_id),
    FOREIGN KEY(branch_id) REFERENCES branch(branch_id) ON DELETE SET NULL
);


CREATE TABLE works_with(
    emp_id INT,
    client_id INT,
    total_sales INT,

    PRIMARY KEY(emp_id, client_id),
    FOREIGN KEY(emp_id) REFERENCES employee(emp_id) ON DELETE CASCADE,
    FOREIGN KEY(client_id) REFERENCES client(client_id) ON DELETE CASCADE
);


CREATE TABLE branch_supplier(
    branch_id INT,
    supplier_name VARCHAR(30),
    supply_type VARCHAR(30),

    PRIMARY KEY(branch_id, supplier_name),
    FOREIGN KEY(branch_id) REFERENCES branch(branch_id) ON DELETE CASCADE
);


-- Corporate
INSERT INTO employee VALUES(100, 'David', 'Wallace', '1967-11-29', 'M', 100000, NULL, NULL);
INSERT INTO branch VALUES(1, 'Corporate', 100, '2000-10-10');


UPDATE employee
SET branch_id = 1
WHERE emp_id = 100;

INSERT INTO employee VALUES(101, 'Jan', 'Levinson', '1988-12-09', 'F', 75000, 100, 1);
INSERT INTO employee VALUES(109, 'Pam', 'Beesly', '1985-02-05', 'F', NULL, 100, 1);
--

-- Scranton
INSERT INTO employee VALUES(102, 'Michael', 'Scott', '1964-03-15', 'M', 75000, 100, NULL);

INSERT INTO branch VALUES(2, 'Scranton', 102, '1992-04-06');

UPDATE employee
SET branch_id = 2
WHERE emp_id = 102;

INSERT INTO employee VALUES(103, 'Angela', 'Martin', '1971-06-25', 'F', 63000, 102, 2);
INSERT INTO employee VALUES(104, 'Kelly', 'Kapoor', '1980-02-05', 'F', 55000, 102, 2);
INSERT INTO employee VALUES(105, 'Stanley', 'Hudson', '1958-02-19', 'M', 69000, 102, 2);

-- Stamford
INSERT INTO employee VALUES(106, 'Josh', 'Porter', '1969-09-05', 'M', 78000, 100, NULL);

INSERT INTO branch VALUES(3, 'Stamford', 106, '1998-02-13');
INSERT INTO branch VALUE(4, 'Buffalo', NULL, NULL);

UPDATE employee
SET branch_id = 3
WHERE emp_id = 106;

INSERT INTO employee VALUES(107, 'Andy', 'Bernard', '1973-07-22', 'M', 65000, 106, 3);
INSERT INTO employee VALUES(108, 'Jim', 'Halpert', '1978-10-01', 'M', 71000, 106, 3);

-- BRANCH SUPPLIER
INSERT INTO branch_supplier VALUES(2, 'Hammer Mill', 'Paper');
INSERT INTO branch_supplier VALUES(2, 'Uni-ball', 'Writing Utensils');
INSERT INTO branch_supplier VALUES(3, 'Patriot Paper', 'Paper');
INSERT INTO branch_supplier VALUES(2, 'J.T. Forms & Labels', 'Custom Forms');
INSERT INTO branch_supplier VALUES(3, 'Uni-ball', 'Writing Utensils');
INSERT INTO branch_supplier VALUES(3, 'Hammer Mill', 'Paper');
INSERT INTO branch_supplier VALUES(3, 'Stamford Labels', 'Custom Forms');

UPDATE branch_supplier
SET supplier_name = 'Stamford Labels'
WHERE branch_id = 3 AND supplier_name = 'Stamford Lables';

-- CLIENT
INSERT INTO client VALUES(400, 'Dunmore Highschool', 2);
INSERT INTO client VALUES(401, 'Lackawana Country', 2);
INSERT INTO client VALUES(402, 'FedEx', 3);
INSERT INTO client VALUES(403, 'John Daly Law, LLC', 3);
INSERT INTO client VALUES(404, 'Scranton Whitepages', 2);
INSERT INTO client VALUES(405, 'Times Newspaper', 3);
INSERT INTO client VALUES(406, 'FedEx', 2);

-- WORKS_WITH
INSERT INTO works_with VALUES(105, 400, 55000);
INSERT INTO works_with VALUES(102, 401, 267000);
INSERT INTO works_with VALUES(108, 402, 22500);
INSERT INTO works_with VALUES(107, 403, 5000);
INSERT INTO works_with VALUES(108, 403, 12000);
INSERT INTO works_with VALUES(105, 404, 33000);
INSERT INTO works_with VALUES(107, 405, 26000);
INSERT INTO works_with VALUES(102, 406, 15000);
INSERT INTO works_with VALUES(105, 406, 130000);

----------------------------------------------------


SELECT first_name AS name, last_name AS surname, salary, sex
FROM employee
ORDER BY sex DESC, salary, first_name ASC
LIMIT 10;


SELECT DISTINCT sex # unikátne hodnoty 
FROM employee;


SELECT COUNT(emp_id) 
# každý emp musí mať id, takže teraz spočítame  počet zamstnancov vo firme
# keď namiesto emp_id napíšeme *, stane sa presne to isté
FROM employee;

SELECT COUNT(emp_id)
FROM employee
WHERE sex = 'F' AND salary > 70000;

SELECT emp_id, first_name, birthdate
FROM employee
WHERE birthdate > '1970-1-1';


SELECT AVG(salary)
FROM employee
WHERE sex = 'M';

SELECT COUNT(sex), sex
FROM employee
GROUP BY sex;


SELECT SUM(total_sales), client_id
FROM works_with
GROUP BY client_id;


SELECT *
FROM client
WHERE client_name LIKE '%LLC';


SELECT *
FROM branch_supplier
WHERE supplier_name LIKE '% Label%';


SELECT *
FROM employee
WHERE birthdate LIKE '____-02-__';


SELECT first_name, emp_id
FROM employee
UNION
SELECT branch_name, branch_id
FROM branch;

SELECT employee.first_name, employee.emp_id, branch.branch_name
FROM employee
JOIN branch  # JOIN, lEFT JOIN, RIGHT JOIN
ON employee.emp_id = branch.manager_id
ORDER BY branch.branch_name DESC, employee.first_name ASC;


SELECT employee.emp_id AS ID, employee.first_name AS employee, works_with.total_sales AS sales, works_with.client_id AS client
FROM employee
JOIN works_with
WHERE works_with.total_sales > 30000 AND employee.emp_id = works_with.emp_id;


SELECT employee.emp_id, employee.first_name
FROM employee
WHERE employee.emp_id IN (
    SELECT works_with.emp_id
    FROM works_with
    WHERE works_with.total_sales > 30000
);


SELECT client.client_name 
FROM client
WHERE client.branch_id = (
    SELECT branch.branch_id
    FROM branch
    WHERE branch.manager_id = 102
    LIMIT 1 # keby bol micheal scott (emp_id = 102) manažerom viacerých prevádzok
);




SELECT * FROM employee;
SELECT * FROM branch;
SELECT * FROM client;
SELECT * FROM works_with;
SELECT * FROM branch_supplier;


DESCRIBE employee;
DESCRIBE branch;
DESCRIBE client;
DESCRIBE works_with;


