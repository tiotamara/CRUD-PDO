# CRUD-PDO
CRUD with PDO
# Basic CRUD operations with PDO 

CRUD = Create, Read, Update, Delete

## Open a database connection

```php
$host = '127.0.0.1';
$dbname = 'test';
$username = 'root';
$password = 'root';
$charset = 'utf8';
$collate = 'utf8_unicode_ci';
$dsn = "mysql:host=$host;dbname=$dbname;charset=$charset";
$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_PERSISTENT => false,
    PDO::ATTR_EMULATE_PREPARES => false,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES $charset COLLATE $collate"
];

$pdo = new PDO($dsn, $username, $password, $options);
```

## Select a single row

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email AND status=:status LIMIT 1");
$stmt->execute(['email' => $email, 'status' => $status]);
$user = $stmt->fetch();
```

## Select multiple rows

With fetch for large results.

```php
$stmt = $pdo->prepare("SELECT * FROM employees WHERE name = :name");
$stmt->execute(['name' => $name]);

foreach ($stmt as $row) {
    // do something with $row
}

// or with the fech method:
while ($row = $stmt->fetch()) {
   // do something with $row
}
```

With fetchAll for small results.

```php
$news = $pdo->query('SELECT * FROM news')->fetchAll();
```

## Insert a single row

```php
$row = [
    'username' => 'bob',
    'email' => 'bob@example.com'
];
$sql = "INSERT INTO users SET username=:username, email=:email;";
$status = $pdo->prepare($sql)->execute($row);

if ($status) {
    $lastId = $pdo->lastInsertId();
    echo $lastId;
}
```

## Insert multiple rows

```php
$rows = [];
$rows[] = [
    'username' => 'bob',
    'email' => 'bob@example.com'
];
$rows[] = [
    'username' => 'max',
    'email' => 'max@example.com'
];

$sql = "INSERT INTO users SET username=:username, email=:email;";
$stmt = $pdo->prepare($sql);
foreach ($rows as $row) {
    $stmt->execute($row);
}
```

## Update a single row

```php
$row = [
    'id' => 1,
    'username' => 'bob',
    'email' => 'bob2@example.com'
];
$sql = "UPDATE users SET username=:username, email=:email WHERE id=:id;";
$status = $pdo->prepare($sql)->execute($row);
```

## Update multiple rows

```php
$row = [
    'updated_at' => '2017-01-01 00:00:00'
];
$sql = "UPDATE users SET updated_at=:updated_at";
$pdo->prepare($sql)->execute($row);

$affected = $pdo->rowCount();
```

## Delete a single row

```php
$where = ['id' => 1];
$pdo->prepare("DELETE FROM users WHERE id=:id")->execute($where);
```

## Delete multiple rows

```php
$pdo->prepare("DELETE FROM users")->execute();
```

## PDO datatypes

Getting dynamic (POST) data with null values can be difficult to handle with PDO. 
Here is a helper function to detect the correct data type.

```php
function get_pdo_type($value)
{
    switch (true) {
        case is_bool($value):
            $dataType = PDO::PARAM_BOOL;
            break;
        case is_int($value):
            $dataType = PDO::PARAM_INT;
            break;
        case is_null($value):
            $dataType = PDO::PARAM_NULL;
            break;
        default:
            $dataType = PDO::PARAM_STR;
    }
    return $dataType;
}

// Usage
$email = $_POST['email'];

$pdo = new PDO('dsn', 'username', 'password');
$sql = 'INSERT INTO users SET email=:email;';
$stmt = $pdo->prepare($sql);
$stmt->bindValue(':email', $email, get_pdo_type($email));
$stmt->execute();
```

## Prepared statements using the IN clause

It cannot be done with PDO, according to the PHP Manual's entry on PDO::prepare(), which says:
"You cannot bind multiple values to a single named parameter in, for example, the IN() clause of an SQL statement."

This PDO helper function converts all array values into a (safe) quoted string. 

```php
function quote_values(PDO $pdo, array $values) {
    array_walk($values, function (&$value) use ($pdo) {
        if($value === null) {
            $value = 'NULL';
            return;
        }
        $value = $pdo->quote($value);
    });
    return implode(',', $values);
}
```

Example usage:

```php
$ids = [
    1,
    2,
    3,
    "'", 
    null,
    'string',
    123.456
];

$sql = sprintf("SELECT id FROM users WHERE id IN(%s)", quote_values($pdo, $ids));
echo $sql . "\n";

$stmt = $pdo->prepare($sql);
$stmt->execute();
```

Generated SQL:

```sql
SELECT id FROM users WHERE id IN('1','2','3','\'',NULL,'string','123.456')
```

Other solutions:

* [PHP PDO Prepared Statements to Prevent SQL Injection](https://websitebeaver.com/php-pdo-prepared-statements-to-prevent-sql-injection#where-in-array)

* [Prepared statements and IN clause](https://phpdelusions.net/pdo#in)

