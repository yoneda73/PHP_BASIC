# PHP基礎 08
## クラスとオブジェクト(2)
### PDOオブジェクト
PDO(PHP Data Objects)はデータベースに接続するためのオブジェクト  
https://www.php.net/manual/ja/book.pdo.php

### インストール
PDOはサーバーにモジュールとして組み込まれている必要があります。  
ユーザーが個別にインストールできるものではありません。  
phpinfo関数で確認が可能です。

### 接続
```
<?php

// ドライバ呼び出しを使用して MySQL データベースに接続します
$dsn = 'mysql:dbname=phpsample;host=127.0.0.1';
$user = 'php';
$password = 'Secret01%';

try {
    $dbh = new PDO($dsn, $user, $password);
    echo "接続成功\n";
} catch (PDOException $e) {
    echo "接続失敗: " . $e->getMessage() . "\n";
    exit();
}
```

* * * 
PDOオブジェクトのコンストラクタ
* * * 
PDOオブジェクトのコンストラクタは三つの引数を必要とします。  
new PDO(<接続情報, ユーザー名, パスワード>);  
接続情報(DSL)には

dbname : データベース名  
host : データベースホスト

を含みます。

###  テストテーブルの作成
```
CREATE TABLE user (
    id int primary key auto_increment, 
    name varchar(50), 
    password varchar(20)
);
INSERT INTO user (name, password) VALUES ('内村鈴子', 'oVg95BO1');
INSERT INTO user (name, password) VALUES ('阿部義勝', 'VAnP6tbW');
INSERT INTO user (name, password) VALUES ('矢島陽菜', 'WQKDwRyp');
INSERT INTO user (name, password) VALUES ('大村勇', 'fuI16Xkv');
INSERT INTO user (name, password) VALUES ('阿部義勝', 'uJf1JivA');
INSERT INTO user (name, password) VALUES ('中西国彦', 'A8x5HzD2');
INSERT INTO user (name, password) VALUES ('古河信明', 'XTu7VDRh');
```

### 一覧を取得
PDO::query()メソッド
SQL ステートメントを実行し、結果セットを PDOStatement オブジェクトとして返す
```
<?php

// ドライバ呼び出しを使用して MySQL データベースに接続します
$dsn = 'mysql:dbname=phpsample;host=127.0.0.1';
$user = 'php';
$password = 'Secret01%';

try {
    //DBに接続
    $dbh = new PDO($dsn, $user, $password);
    
    //SQL
    $sql = 'select id, name, password from user';

    //実行
    $sth = $dbh->query($sql);

    //取得
    foreach ($sth as $row) {
        print $row['id'] . "\t";
        print $row['name'] . "\t";
        print $row['password'] . PHP_EOL;
    }

} catch (PDOException $e) {
    echo "接続失敗: " . $e->getMessage() . "\n";
    exit();
}

//MySQL切断
$dbh = null;
```

### プリペアドステートメント
SQL文を変数で調整したい場合に利用します。
```
<?php

// ドライバ呼び出しを使用して MySQL データベースに接続します
$dsn = 'mysql:dbname=phpsample;host=127.0.0.1';
$user = 'php';
$password = 'Secret01%';

try {
    //接続
    $dbh = new PDO($dsn, $user, $password);

    //登録情報
    $username = "国森奈々穂";
    $password = "abcd1234";

    //SQL
    $sql = 'INSERT INTO user (name, password) VALUES(?,?)';
    //PreparedStatment
    $stmt = $dbh->prepare($sql);
    $stmt->bindParam(1, $username);
    $stmt->bindParam(2, $password);
    $result = $stmt->execute();

    var_dump($result);
    
} catch (PDOException $e) {
    echo "接続失敗: " . $e->getMessage() . "\n";
    exit();
}
```
POD prepare クラス  
https://www.php.net/manual/ja/pdo.prepare.php  
PDO::statment  
https://www.php.net/manual/ja/class.pdostatement.php  

```
//SQL
$sql = 'INSERT INTO user (name, password) VALUES(:name, :password)';
/**略**/
$stmt->bindParam(':name', $username);
$stmt->bindParam(':password', $password);
```
* * * 
execute()メソッドの戻り値
* * * 
execute()メソッドの戻り値は真偽値(boolean)  
成功すればtrue、失敗するとfalse。

### 練習問題
Formから値を取得してデータベースに構築するアプリケーションの作成

| 項目    | データ型  | 備考            | 
| ------- | --------- | --------------- | 
| book_id | int       | 主キー 自動連番 | 
| title   | varchar(100) | 本のタイトル    | 
| author  | varchar(50) | 著者           | 
| publisher  | varchar(50) | 出版社       | 
| price  | int | 価格            | 

#### 構成
- book_list.php(本の一覧)
- book_form.php(本の情報追加)
- book_form_confirm.php(本の情報追加確認)
- book_form_complete.php(本の情報追加完了)

#### DBの準備
DB(phpsample)に新しいテーブル(books)を作成。  
数件、テストデータを入れて準備をします。  
```
CREATE TABLE books (
    book_id int primary key auto_increment,
    title varchar(100),
    author varchar(50),
    publisher varchar(50),
    price int
);
INSERT INTO books(title,author,publisher,price) VALUES ("PHP本格入門［上］　～プログラミングとオブジェクト指向の基礎からデータベース連携まで","大家 正登","技術評論社",3938);
INSERT INTO books(title,author,publisher,price) VALUES ("よくわかるPHPの教科書　【PHP7対応版】","たにぐち まこと","マイナビ出版",2728);
```

#### 一覧表示のページを作成
book_list.php
```
<?php
$dsn = 'mysql:dbname=phpsample;host=127.0.0.1';
$user = 'php';
$password = 'Secret01%';

try {
    //DBに接続
    $pdo = new PDO($dsn, $user, $password);
    
    //SQL
    $sql = 'select book_id, title, author, publisher, price from books';

    //実行
    $sth = $pdo->query($sql);

} catch (PDOException $e) {
    echo "接続失敗: " . $e->getMessage() . "\n";
    exit();
}
?>
<!DOCTYPE html>
<html lang="ja".....
```
表示したい部分で展開する
```
<table>
<?php foreach($sth as $row): ?>
<tr>
<td><?=$row['book_id']?></td>
<td><?=$row['title']?></td>
<td><?=$row['author']?></td>
<td><?=$row['publisher']?></td>
<td><?=$row['price']?></td>
</tr>
<?php endforeach ?>
</table>
```
