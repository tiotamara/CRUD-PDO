# CRUD-PDO
CRUD with PDO
# Basic CRUD Operasi dengan PDO 

CRUD = Create, Read, Update, Delete

## Untuk file koneksi yang akan dikoneksikan ke database.

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

## Mengambil data row.

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email AND status=:status LIMIT 1");
$stmt->execute(['email' => $email, 'status' => $status]);
$user = $stmt->fetch();
```

## Mengambil data multiple.

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

## Dengan fetchAll.

```php
$news = $pdo->query('SELECT * FROM news')->fetchAll();
```

## Memasukkan single data.

```php
$row = [
    'username' => 'tio',
    'email' => 'tio@ganteng.com'
];
$sql = "INSERT INTO users SET username=:username, email=:email;";
$status = $pdo->prepare($sql)->execute($row);

if ($status) {
    $lastId = $pdo->lastInsertId();
    echo $lastId;
}
```

## Memasukkan Data Multiple.

```php
$rows = [];
$rows[] = [
    'username' => 'tio',
    'email' => 'tio@ganteng.com'
];
$rows[] = [
    'username' => 'tioganteng',
    'email' => 'tioganteng@max.com'
];

$sql = "INSERT INTO users SET username=:username, email=:email;";
$stmt = $pdo->prepare($sql);
foreach ($rows as $row) {
    $stmt->execute($row);
}
```

## Update data.

```php
$row = [
    'id' => 1,
    'username' => 'tio',
    'email' => 'tio@ganteng.com'
];
$sql = "UPDATE users SET username=:username, email=:email WHERE id=:id;";
$status = $pdo->prepare($sql)->execute($row);
```

## Update multiple data.

```php
$row = [
    'updated_at' => '2017-01-01 00:00:00'
];
$sql = "UPDATE users SET updated_at=:updated_at";
$pdo->prepare($sql)->execute($row);

$affected = $pdo->rowCount();
```

## Delete data

```php
$where = ['id' => 1];
$pdo->prepare("DELETE FROM users WHERE id=:id")->execute($where);
```

## Delete multiple data

```php
$pdo->prepare("DELETE FROM users")->execute();
```

## PDO datatypes

Mendapatkan data dinamis (POST) dengan nilai null bisa sulit ditangani dengan PDO.
Berikut adalah fungsi pembantu untuk mendeteksi tipe data yang benar.

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

## Prepared statements dengan IN klause

Fungsi pembantu PDO ini mengubah semua nilai array menjadi string yang dikutip (aman).

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

