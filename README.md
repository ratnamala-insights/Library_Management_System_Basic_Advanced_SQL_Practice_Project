# Library_Management_System_Basic_Advanced_SQL_Practice_Project

**Project Overview**
This project is a SQL-based Library Management System developed for hands-on practice of relational database design and query writing. The goal of this project was to strengthen practical knowledge of:

Database schema design
Primary & foreign key relationships
CRUD operations
Joins and aggregations
Stored procedures
Business logic implementation in SQL
Reporting queries

**Database Schema & ER Diagram**

<img width="602" height="334" alt="Library_schema" src="https://github.com/user-attachments/assets/3a27d042-686a-4c09-8536-90c81e4ad38c" />

The database consists of 6 interconnected tables:

**1. Branch**
Primary Key: branch_id
**Columns:**
branch_id
manager_id
branch_address
contact_no

**2. Employees**

Primary Key: emp_id
Foreign Key: branch_id → Branch(branch_id)
**Columns:**
emp_id
emp_name
position
salary
branch_id

**3. Members**

Primary Key: member_id
**Columns:**
member_id
member_name
member_address
reg_date

**4. Books**

Primary Key: isbn
**Columns:**
isbn
book_title
category
rental_price
status
author
publisher

**5. Issued_Status**
Primary Key: issued_id
Foreign Keys:
issued_member_id → Members(member_id)
issued_book_isbn → Books(isbn)
issued_emp_id → Employees(emp_id)
**Columns:**
issued_id
issued_member_id
issued_book_name
issued_date
issued_book_isbn
issued_emp_id

**6. Return_Status**

Primary Key: return_id
Foreign Key: issued_id → Issued_Status(issued_id)
**Columns:**
return_id
issued_id
return_book_name
return_date
return_book_isbn

**SQL Practice Questions**

**--Update an existing member's address**
```sql
update members 
set member_address = '125 Main St'
where member_id = 'C101';
```

**--Delete the record with issued_id = 'IS122' from the issued_status table**
```sql
delete from issued_status
where issued_id = 'IS122';
```
**--Select all the books issued by Employee with employee_id: 'E101'**
```sql
Select issued_book_name
from issued_status
where issued_emp_id = 'E101';
```
**--Use Group by to find member swho have issued more than one book**
```sql
Select issued_emp_id, count(issued_emp_id)
from issued_status
group by issued_emp_id
having count(issued_emp_id) > 1;
```
**--Find each book and total issued count**
```sql
select Books.book_title, count(issued_status.issued_id) as no_issued
from Books
Join issued_status
on books.isbn = issued_status.issued_book_isbn
group by Books.book_title;
```
**-- Retrive all books in a specific category - "classic"**
```sql
Select category,book_title
from Books
where category = 'classic'
```
**-- Find total Rental Income by category**
```sql
Select category, sum(rental_price) as total_rental_income
from Books
group by category
order by sum(rental_price) desc;
```
**--List Members who registered in the last 180 days**
```sql
Select * from members
where reg_date >= DATEADD(DAY, -180, GETDATE())
```
**--list employees with their Branch manager's Name and their branch details:**
```sql
Select e1.Emp_id, e1.Emp_name, branch.manager_id, e2.Emp_name as Manager_name
from Employees e1
Join branch
on e1.branch_id = branch.branch_id
join Employees e2
on Branch.manager_id = e2.emp_id;
```
**--Create a table of book with rental price above $6**
```sql
Select * Into books_price_6
from Books
where rental_price > 6;

**--Check the table**
Select * from books_price_6;
```
**--Retrieve the list of books not yet returned**
```sql
Select issued_status.issued_book_name, issued_status.issued_id
from issued_status
Left join return_status
on issued_status.issued_id = return_status.issued_id
where return_status.return_id is Null;
```
**--Advanced Questions--**

**--Adding new column in return_status -Book Quality**
```sql
Alter Table return_status
Add book_quality varchar(15) Default 'Good';
```
**--update column book_quality columns for entries issue_id IS112, IS117, IS118 as damaged**
```sql
Update return_status
set book_quality = 'damaged'
where issued_id in ( 'IS112', 'IS117', 'IS118');
```
**--Set the book_quality to good for Null**
```sql
Update return_status 
set book_quality = 'Good' 
where book_quality IS NULL;
```

**--Identify members with overdue books
Write a query to identify members who have overdue books( assume a 30 day return period). Display the member's id, member's name, Book_title, issue date & days overdue**
```sql
select ist.issued_member_id,m.member_name,b.book_title,ist.issued_date, rst.return_date,
DATEDIFF(DAY, ist.issued_date, GETDATE()) AS over_due_days
from members m
join issued_status ist
on m.member_id = ist.issued_member_id 
join books b
on ist.issued_book_isbn = b.isbn
Left join return_status rst   --as we need to check all the records ie. 34 records 
on rst.issued_id = ist.issued_id
where rst.return_date is null  -- all the records having return_date as null
and DATEDIFF(DAY, ist.issued_date, GETDATE()) > 30
order by over_due_days desc;
```
**Update book status on return
Write a query to update the status of books in the books table to "yes" when they are returned(based on entries in the return_status table)**

**--First make the status of one of the book to No in books table for later work**

update Books
set status = 'No'
where isbn = '978-0-451-52994-2';

**--Check if successfull**
Select * from Books; -- Yes

**-- if we want to solve the query manually, we have to follow all the below steps
--First we will update return status table**
Insert into return_status(return_id, issued_id, return_date, book_quality)
values('RS125','IS130', CAST(GETDATE() as date), 'Good')

**--Check if the records are updated**
Select * from return_status
where issued_id = 'IS130';

**--Now we have to again change the status of the book status to 'Yes' which is appearing as 'NO'**

Update Books
set status = 'Yes'
where isbn = '978-0-451-52994-2';

**-- check if changed**
Select * from Books
where isbn = '978-0-451-52994-2';-- changed

****--So we have to follow all the above steps manually hence we need **Stored procedures****
```sql
Create Procedure Update_return_book_status (@return_id varchar(10), @issued_id Varchar(10), 
 @return_date date,@book_quality varchar(15))
 
 As 
 Begin
 --Insert into return_status
	Insert into return_status(return_id, issued_id, return_date, book_quality)
	values(@return_id, @issued_id,  @return_date,@book_quality)

 -- Update the book status to 'Yes'
	Update b
	set b.status = 'Yes'
	from books b
	join issued_status.ist
	on b.isbn = ist.issued_book_isbn
	where ist.issued_id = @issued_id; --as we only know issued_id while returning the book
	  
 End

 Exec Update_return_book_status 'RS104','IS106', '2026-02-20','Good';
```

 **Create a query that generates a performane report for each branch,Showing the number of books issued,the number of books returned, & the total revenue generated from book rentals**
 ```sql
Select 
 br.branch_id, 
 count(ist.issued_id) as 'books_issued', 
 count(rst.return_id) as 'returned_books', 
 sum(bk.rental_price) as 'rental_price'
 into branch_permance_report -- save the summary of table
from 
	Branch br
	join Employees emp
	on br.branch_id = emp.branch_id
	join issued_status ist
	on emp.emp_id = ist.issued_emp_id
	Left join return_status rst
	on ist.issued_id = rst.issued_id
	Left join books bk
	on ist.issued_book_isbn = bk.isbn
group by br.branch_id;

Select * from branch_permance_report; 
```
**Create a Summary table of Active members
Use a create table as CTAS statement to create a new table active_members containing members who have issued atleast 1 book in last 2 months**

```sql
Select issued_member_id, count(issued_id) as count_books_last_2_months
into active_members
from
issued_status
where issued_date >= DATEADD(month, -2,GETDATE()) -- records from last 2 months till today
group by issued_member_id

Select * from active_members;
```
**Find Employees with the most Book issued processed. Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed and their branch**
```sql
Select Top 3 emp.branch_id,emp.emp_name,count(ist.issued_id)
from Employees emp
join issued_status ist
on emp.emp_id = ist.issued_emp_id
group by emp.branch_id,emp.emp_name
order by count(ist.issued_id) desc;
```
**Create a Stored procedure to manage the status of books in a library system, updates the status
of a book in the library based on its issuance. Operation: The Stored procedure should take the book_id as an input parameter. 
The Procedure should first check if the book is available(status = 'yes'). If the book is available, it should be issued, and the status in the books table should be updated to 'No'. If the book is not available(status='no'). the procedure should return an error message indicating that the book is currently not available**

**/*logic**
Create procedure manage_status @isbn varchar(25),
							   @issued_id Varchar(10),
							   @issued_member_id Varchar(10),
						       @issued_date date,
							   @issued_book_isbn Varchar(25),
							   @issued_emp_id Varchar(10)
As
begin
	--Checking if the book is available ie. status= 'yes'
	Select status 
	from books
	where isbn = @isbn;

	if
	@isbn = yes
       then
			insert into issued_status(issued_id, issued_member_id,issued_date,issued_book_isbn, issued_emp_id) 
			Values( @issued_id,@issued_member_id,issued_date,issued_book_isbn,
					  issued_emp_id);

	Update Books
	Set status = 'No'
	where isbn = @isbn
	
      print 'Book with' @isbn 'is added sucessfully in records';
	else
	   
	    print'Sorry to inform but the Book is unavailable'

End
*/

```sql
CREATE PROCEDURE manage_status
    @isbn VARCHAR(25),
    @issued_id VARCHAR(10),
    @issued_member_id VARCHAR(10),
    @issued_date DATE,
    @issued_emp_id VARCHAR(10)
AS
BEGIN

    DECLARE @book_status VARCHAR(10);

    -- Get current book status
    SELECT @book_status = status
    FROM books
    WHERE isbn = @isbn;

    -- Check if book is available
    IF @book_status = 'yes'
    BEGIN
        -- Insert into issued_status table
        INSERT INTO issued_status
            (issued_id, issued_member_id, issued_date,
             issued_book_isbn, issued_emp_id)
        VALUES
            (@issued_id, @issued_member_id, @issued_date,
             @isbn, @issued_emp_id);

        -- Update book status to 'No'
        UPDATE books
        SET status = 'No'
        WHERE isbn = @isbn;

        PRINT 'Book with ' + @isbn + ' is issued successfully.';
    END
    ELSE
    BEGIN
        PRINT 'Sorry, the book is currently unavailable.';
    END

END;
```
**Conclusion**
This project demonstrates the ability to translate business requirements into SQL queries and implement structured relational database logic using using Microsoft SQL Server.
