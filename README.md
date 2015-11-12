# anytv-node-mysql

Our version of mysql that makes connecting to mysql simpler and more elegant. Especially made for our awesome expressjs [boilerplate](https://github.com/anyTV/anytv-node-boilerplate).


# Install

```sh
npm install anytv-node-mysql --save
```

# Features

* Add a key-config pair once and use it anywhere
* Chain everything
* Do transactions elegantly
* Automatic rollback on transactions
* Easy to specify if you want a pool connection or a per-query connection
* Specify errors that can be retried and set their max retries
* Auto-reconnect
* Easy debugging
* Use `.args(...)` to help closures


# Usage

### Adding database config
On your index.js / server.js / app.js, register your database using a key.
```javascript
import mysql from 'anytv-node-mysql';

mysql.add('my_db', {
	host: 'localhost',
	user: 'root',
	password: '',
	database: test
});
```


### Doing a single query
After registering a db key and config, you can now start querying.
```javascript
mysql.query(
		'SELECT name FROM users WHERE name = ?',
		['name'],
		callback
	)
	.end();
```

**Note:** Single connections will only be created on `mysql.query(...)` while pooled connections will be created on `mysql.use('db1', config.DB, true)`.

### Doing multiple queries using 1 connection
You can chain queries and use a single connection.
```javascript
mysql.query(
		'SELECT name FROM users WHERE name = ?',
		['name'],
		callback
	)
	.query(
		'SELECT address FROM users WHERE name = ?',
		['name2'],
		callback2
	)
	.end();
```

### Using multiple database configs
In case you registered multiple databases, you can switch the current db by invoking `mysql.use(key)`.
```javascript
import mysql from 'anytv-node-mysql';

mysql.add('db_1', config.DB1);
mysql.add('db_2', config.DB2);

mysql.use('db1')
	.query(
		'SELECT name FROM users WHERE name = ?',
		['name'],
		callback
	)
	.end();


mysql.use('db2')
	.query(
		'SELECT name FROM users WHERE name = ?',
		['name2'],
		callback2
	)
	.end();
```

You can also switch databases right after ending.
```javascript
mysql.use('db1')
	.query(
		'SELECT name FROM users WHERE name = ?',
		['name'],
		callback
	)
	.end() // if you forgot to call this, the .use('db2') will do it for you
	.use('db2')
	.query(
		'SELECT name FROM users WHERE name = ?',
		['name2'],
		callback2
	)
	.end();
```

### Doing transactions
Creating a transaction returns a transaction object, not the mysql object. It's not possible to do another query after doing a commit.
```javascript
mysql.use('db1')
	.transaction()
	.query('INSERT INTO unique_email(email) VALUES (?)', ['unique'], callback)
	.query('INSERT INTO unique_email(email) VALUES (?)', ['unique'], callback)
	.query('SELECT * FROM unique_email WHERE email = ?', ['unique'], callback)
	.commit(final_callback);
```
If a query fails, the next queries won't be executed anymore. Plus, the `callback` and `final_callback` will have the same arguments.


### Solving problem with closures
Same with you, we had trouble with closures when we're querying inside a loop. We created a function where you can pass anything and we'll include it on the callback.
```javascript
mysql.use('db1')
	.args(obj, 2, 'config')
	.query('INSERT INTO unique_email(email) VALUES (?)', ['unique'], callback)
	.end();

function callback (err, result, args, last_query) {
	args[0] === obj;
	args[1] === 2;
	args[2] === 'config';
}
```

### Handling callbacks and debugging errors
We follow the node convention of callbacks. Error first followed by result.
```javascript
mysql.use('db1')
	.query('INSERT INTO unique_email(email) VALUES (?)', ['unique'], callback)
	.end();

function callback (err, result, args, last_query) {
	if (err) {
		// do something with the error
		...
		return;
	}

	// do something with the result
	...
}
```
The last executed query is accessible on the `last_query` variable.


### Setting retryable errors
To solve intermittent issues, we added a function that accepts an array of error codes where the library will keep on retrying until it works or until it reaches the maximum retries. Default maximum retries is 3.
```javascript
mysql.use('db1')
	.retry_if(['ER_LOCK_DEADLOCK', 'PROTOCOL_SEQUENCE_TIMEOUT'])
	.set_max_retry(5)
	.query('INSERT INTO unique_email(email) VALUES (?)', ['unique'], callback)
	.end();
```
**NOTE:** We clear the retryable errors on `mysql.use(key)`.


### Solving on-the-fly db config
We added a `.open` function that accepts a config.
```javascript
mysql.open({
		host: 'localhost',
		user: dynamic_variable,
		password: dynamic_variable2,
		database: 'test'
	})
	.query('INSERT INTO unique_email(email) VALUES (?)', ['unique'], callback)
	.end();
```


### Instantiating a pooled connection
The `mysql.use(...)` accepts a boolean 3rd parameter. If true, we'll use a pool connection and create it immediately. Calling `.end()` on pooled connections won't do anything.
```javascript
mysql.add('db1', config.DB1, true);

mysql.use('db1')
	.query('INSERT INTO unique_email(email) VALUES (?)', ['unique'], callback)
	.end();
```


### Using a different logger
```javascript
mysql.set_logger(my_new_logger);
```




# Contributing

Install the tools needed:
```sh
npm install babel -g
npm install babel-preset-es2015 --save-dev
npm install esdoc -g
npm install --dev
```

To compile the ES6 source code to ES5:
```sh
babel src --watch --out-dir lib
```

To generate the docs:
```sh
esdoc -c ./esdoc.json
```

# Running test

```sh
npm test
```

# License

MIT


# Author
[Freedom! Labs, any.TV Limited DBA Freedom!](https://www.freedom.tm)