
## How to prevent sql injection vulnerabilities how prepared statements work

- [How to prevent sql injection vulnerabilities how prepared statements work](#how-to-prevent-sql-injection-vulnerabilities-how-prepared-statements-work)
- [What are Prepared Statements?](#what-are-prepared-statements)
- [**How Prepared Statements work?**](#how-prepared-statements-work)
- [**Examples for different languages**](#examples-for-different-languages)
  - [**Java**](#java)
  - [**C#**](#c)
  - [**PHP**](#php)
  - [**Python**](#python)
  - [**Go**](#go)
  - [**Ruby**](#ruby)
  - [**Node**](#node)
- [**Alternatives to Prepared Statements**](#alternatives-to-prepared-statements)
- [**Stored Procedures**](#stored-procedures)
  - [**Allowlist**](#allowlist)
  - [**What about input sanitization?**](#what-about-input-sanitization)
- [**Questions Here Okay Cool**](#questions-here-okay-cool)


**Introduction**


## What are Prepared Statements?



https://www.hackedu.com/blog/how-to-prevent-sql-injection-vulnerabilities-how-prepared-statements-work

1123

SQL Injection is a software vulnerability that occurs when user-supplied data is used as part of a SQL query. Due to improper validation of data, an attacker can submit a valid SQL statement that changes the logic of the initial query used by the application. As a result, the attacker can view/modify/delete sensitive data of other users or even get unauthorized access to the entire system.

While easy to fix, SQL Injection vulnerabilities are still prevalent. In this article, we will discuss how to prevent these vulnerabilities through good coding practices. We will focus on prepared statements, how they work, and how you can implement them.


A prepared statement is a parameterized and reusable SQL query which forces the developer to write the SQL command and the user-provided data separately. The SQL command is executed safely, preventing SQL Injection vulnerabilities.

Here is an example of an unsafe approach in PHP:

$query = "SELECT * FROM users WHERE user = '$username' and password = '$password'";

$result = mysql_query($query);

As you can see, the user-provided data is embedded directly in the SQL query. If the user inserts **admin** and **a' or '1'='1**, she will be able to login to the admin account without knowing the password because the SQL statement has been altered.

Here is an example of a prepared statement approach in PHP:

$stmt = $mysqli->prepare("SELECT * FROM users WHERE user = ? AND password = ?");

$stmt->bind_param("ss", $username, $password);

$stmt->execute();

The user-supplied data is not directly embedded in the SQL query in this example. Instead of the user’s data there is a ? symbol. That is a placeholder and temporarily takes the place of the data. The SQL query is pre-compiled with placeholders, and the user’s data is added later. If the user inserts **admin** and **a' or '1'='1**, the initial SQL query logic won’t be changed. Instead, the database will look for a user **admin** whose password is literally **a' or '1'='1**.

## **How Prepared Statements work?**

Before discussing how prepared statement works, let’s have a look at the SQL query processing workflow:

<div>![fig1](https://www.hackedu.com/hubfs/fig1.png)</div>

_Fig 1: Oversimplified representation of SQL query processing_

As you can see, the process involves six steps:

1.  **Parsing** - The SQL query is broken into individual words (also called tokens). Syntax error and misspelling checks are performed to ensure the validity of the SQL query.
2.  **Semantics Check** - The Database Management System (DBMS) establishes the validity of the query. Does the specified columns and table exist? Does the user have privileges to execute this query?
3.  **Binding** - The query is converted into a format understandable by machines: byte code. Next, the query is compiled and sent to the database server for optimization and execution.
4.  **Query Optimization** - The DBMS chooses the best algorithm for executing the query, considering the cost.
5.  **Cache** - The best algorithm is saved in the cache, so next time when the same query is executed it will skip the first four steps and jump straight to the execution.
6.  **Execution -** The query is executed and the results are returned to the user.

But how does a prepared statement go through this process since they are different from a normal query?

The process is similar, but with a few differences:

1.  **Parsing** and **Semantics Check** are the same.
2.  With **Binding**, the database engine detects the placeholders, and the query is compiled with **placeholders**. The user-supplied data will be added later.
3.  The **Cache** step is the same. The query is stored in cache for further use.
4.  Between **Cache** and **Execution**, there is an additional step: **Placeholder Replacement**. At this point, the placeholders are replaced with the user’s data. However, the query is already pre-compiled (**Binding**), so the final query will not go through compilation phase again. For this reason, the user-provided data will always be interpreted as a simple string and cannot modify the original query’s logic. Thus, the query will be immune to SQL Injection vulnerabilities for that data.

<div>![fig2](https://www.hackedu.com/hubfs/fig2.png)</div>

_Fig 2\. Oversimplified representation of SQL prepared statements processing_

## **Examples for different languages**

### **Java**

The JDBC API has a class called PreparedStatement that can be used to safely handle user input as part of an SQL command. Here are a few examples:

**SELECT Statement** (Source: [https://bobby-tables.com/java](https://bobby-tables.com/java))

String name = //user input

int age = //user input

Connection connection = DriverManager.getConnection(...);

PreparedStatement statement = connection.prepareStatement(

"SELECT * FROM people WHERE lastName = ? AND age > ?" );

statement.setString(1, name); //lastName is a VARCHAR

statement.setInt(2, age); //age is an INT

ResultSet rs = statement.executeQuery();

while (rs.next()){

//...

}

**UPDATE Statement** (Source: [https://bobby-tables.com/java](https://bobby-tables.com/java))

List<Person>; people = //user input

Connection connection = DriverManager.getConnection(...);

connection.setAutoCommit(false);

try {

PreparedStatement statement = connection.prepareStatement(

"UPDATE people SET lastName = ?, age = ? WHERE id = ?");

for (Person person : people){

statement.setString(1, person.getLastName());

statement.setInt(2, person.getAge());

statement.setInt(3, person.getId());

statement.execute();

}

connection.commit();

} catch (SQLException e) {

connection.rollback();

}

**INSERT Statement** (Source: [https://alvinalexander.com/blog/post/jdbc/create-use-preparedstatement](https://alvinalexander.com/blog/post/jdbc/create-use-preparedstatement))

String query = "INSERT INTO Users ("

+ " user_id,"

+ " username,"

+ " firstname,"

+ " lastname,"

+ " companyname,"

+ " email_addr,"

+ " want_privacy ) VALUES ("

+ "null, ?, ?, ?, ?, ?, ?)";

try {

// set all the preparedstatement parameters

PreparedStatement st = conn.prepareStatement(query);

st.setString(1, user.getName());

st.setString(2, user.getFirstName());

st.setString(3, user.getLastName());

st.setString(4, user.getCompanyName());

st.setString(5, user.getEmail());

st.setString(6, user.getPrivacy());

// execute the preparedstatement insert

st.executeUpdate();

st.close();

}

catch (SQLException se)

{

// log exception

throw se;

}

### **C#**

Note: In **C#** the placeholder is not the ? symbol, but @* (where * can be any string you want).

**SELECT Statement** (Source: https://stackoverflow.com/questions/11070434/using-prepared-statement-in-c-sharp-with-mysql)

cmd = new MySqlCommand("SELECT * FROM admin WHERE admin_username=@val1 AND admin_password=PASSWORD(@val2)", MySqlConn.conn);

cmd.Parameters.AddWithValue("@val1", tboxUserName.Text);

cmd.Parameters.AddWithValue("@val2", tboxPassword.Text);

cmd.Prepare();

**INSERT Statement** (Source: [https://downloads.mysql.com/docs/connector-net-en.pdf](https://downloads.mysql.com/docs/connector-net-en.pdf)

string sql = "INSERT INTO foo VALUES(NULL, @number, @text)";

MySqlCommand cmd = new MySqlCommand(sql, conn);

cmd.Parameters.Add("@number", i);

cmd.Parameters.Add("@text", "A string value");

cmd.Prepare();

MySqlDataReader rdr = cmd.ExecuteReader();

### **PHP**

**SELECT Statement**

$stmt = $db->prepare('SELECT * FROM users where name = ? where id = ?');

$stmt->bind_param(‘si’, $name, $id);

$stmt->execute();

**INSERT Statement**

$stmt = $db->prepare("INSERT INTO foo (firstname, lastname, email) VALUES (?, ?, ?)");

$stmt->bind_param("sss", $firstname, $lastname, $email);

$stmt->execute();

### **Python**

If you are using MySQL or PostgreSQL, use %s (even for numbers and other non-string values!) and if you are using SQLite use ?. If you are using ODBC to connect to the DB, regardless of which DB it is, use ?. ([https://bobby-tables.com/python](https://bobby-tables.com/python))

**SELECT Statement**

c.execute("SELECT * FROM foo WHERE bar = %s AND baz = %s", (param1, param2))

### **Go**

**SELECT Statement** (Source: [https://bobby-tables.com/go](https://bobby-tables.com/go))

age := 27

rows, err := db.Query("SELECT name FROM users WHERE age=?", age)

### **Ruby**

**SELECT Statement** (Source: [https://bobby-tables.com/ruby](https://bobby-tables.com/ruby)

Person.find :all, :conditions => ['id = ? or name = ?', id, name]

Or

Person.find_by_sql ['SELECT * from persons WHERE name = ?', name]

### **Node**

**SELECT Statement** (Source: [https://github.com/mysqljs/mysql#escaping-query-values](https://github.com/mysqljs/mysql#escaping-query-values))

connection.query('SELECT * FROM users WHERE id = ?', [userId], function (error, results, fields) {

if (error) throw error;

// ...

});

**INSERT Statement** (Source: [https://github.com/mysqljs/mysql#escaping-query-values](https://github.com/mysqljs/mysql#escaping-query-values))

var post = {id: 1, title: 'Hello MySQL'};

var query = connection.query('INSERT INTO posts SET ?', post, function (error, results, fields) {

if (error) throw error;

// Neat!

});

## **Alternatives to Prepared Statements**

## **Stored Procedures**

When implemented correctly, stored procedures produce the same results as prepared statements. The difference between them is that SQL code for a stored procedure is saved in the database itself and called from the application, while prepared statements are “prepared” every time you need to execute one.

Both of them offer the same level of effectiveness, so it’s up to you which one to use. However, if you choose stored procedures, make sure they do not include any unsafe dynamic SQL generation.




### **Allowlist**

If your application has a feature where a user can choose an option from a finite number of options (such as the type of account (business / personal), the sort order indicator (asc/desc), etc.), then you can use a allowlist approach. A user can only select from options and cannot supply their own.

Here is an example:

switch($accType) {

case “business”:

// execute query for business account

break;

case “personal”:

// execute query for personal account

break;

default:

echo “Unexpected value provided. Please choose from business/personal”;

}

However, this approach is prone to error. It is easy for a developer to implement it in a wrong way, making the application vulnerable to SQL Injection. Thus, use this option in conjunction with prepared statements or stored procedures.

### **What about input sanitization?**

Input sanitization is the process of removing any unwanted characters from user-supplied data(e.g., ‘ / “ { } ). However, this approach is insufficient to prevent all types of SQL Injection and very difficult to get right. There are a lot of bypass techniques such as encoding data to get around sanitization. Try to avoid using input sanitization as much as possible. Instead, use prepared statements or stored procedures.

If you found this post helpful, you may also be interested in reading [our article on Same-Origin Policy and Cross Origin Resource Sharing (CORS).](/blog/same-origin-policy-and-cross-origin-resource-sharing-cors)

</span>

## **Questions Here Okay Cool**