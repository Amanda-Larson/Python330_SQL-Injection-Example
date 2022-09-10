You should be able to start the exercise with the following commands:

- $: cd sql_injection_example
- $: mkvirtualenv sql_injection_example
- $: pip install -r requirements.txt 
- $: python setup.py
- $; python main.py

You should then be able to visit the application by pointing your browser to http://localhost:6779 (Links to an external site.). Now, attempt to craft a message such that the log will show a message posted by the IP address 256.256.256.256: an impossible IP address.

There are multiple ways to approach this exercise, some of which will not work in the SQLite dialect of database queries.  Here is an approach that I recommend: do a search for inserting multiple database rows in a single SQL statement. Then consider the following code, which is responsible for inserting a message into the database:

        transaction = "INSERT INTO messages VALUES ('{}', '{}')".format(
            request.remote_addr,
            request.form['content'],
        )
The format method inserts the client's address into the first set of curly braces, and the message content into the second set of curly braces. Try to craft a message content such that, when it is inserted into the second set of curly braces, the resulting transaction string is a statement which inserts two sets of values instead of just one.

How can you prevent a database injection attack? I implement a very simple solution:

        transaction = "INSERT INTO messages VALUES ('{}', '{}')".format(
            request.remote_addr,
            request.form['content'],
        ).strip("'")
The attacker in this scenario has taken advantage of the fact that the single quote (') is a control character in SQL statements. By putting a single quote into the content of the message that they are posting to the message board, the attacker was able to "break out" of the SQL syntax enclosing the message content and trick the program into executing arbitrary SQL syntax.

The solution involved stripping out all single quotes from the submitter's input. This prevented the attacker from "breaking out" of the SQL syntax. A slightly more sophisticated solution would involve replacing the single quote with an appropriate escape sequence: just as we sometimes replace a single quote in Python with a \' escape sequence, in SQL syntax a single quote can be escaped using two single quotes: ''. If we replaced all single quotes in an submitter's input with double single quotes, then SQL would store those single quotes without treating them as control characters.

Having defeated the "single quote" attack, you might wonder if an attacker might use some other characters to execute a database injection attack. Indeed, strategies involving characters other than the single quote may be viable. Some languages provide sanitization functions. A sanitization function for SQL statements would receive a string (such as a the message content submitted by a user) and return a "sanitized" string which could be safely included in an INSERT statement. Such a string would have any single quotes removed, or replaced with an escape sequence, along with any other potentially dangerous characters.

The standard for safely interacting with databases using Python, as well as some other languages, is to use prepared statements. Programmers create typed functions for interacting with databases, instead of manually crafting statement strings.  A function for inserting a message into our message board database would accept a string for the client IP address and a string for the message content: the function would know that each of these are meant to represent a single cell in the database and would not allow either to have an effect spanning multiple rows: injection attacks become nearly impossible with prepared statements.

Object-relational mappers (ORMs) almost universally use prepared statements. 