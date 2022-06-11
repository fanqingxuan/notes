### PHP
### 介绍

使用过Laravel框架的同学应该都用过 php artisan migrate命令，这个命令是进行数据库迁移的，个人特别喜欢，通过翻阅各种资料，发现了Phinx这个红宝石，可以在自己的项目中直接使用的数据库迁移工具。Phinx是一个数据库迁移工具，用数据库迁移工具的优点，个人总结如下

- 避免了手写sql语句
- 支持多种数据库系统之间的迁移
- 引入版本了概念，已经执行过的会自动跳过
- 可以进行回滚

### Phinx环境要求

PHP 7.2+

### 安装

```shell
composer require robmorgan/phinx
```

### 配置

- 项目根目录执行命令,生成配置文件phinx.php,进行数据库的配置

  ```php
   ./vendor/bin/phinx init
  ```

  phinx.php文件代码如下

  ```php
  <?php
  
  return
  [
      'paths' => [
          'migrations' => '%%PHINX_CONFIG_DIR%%/db/migrations',
          'seeds' => '%%PHINX_CONFIG_DIR%%/db/seeds'
      ],
      'environments' => [
          'default_migration_table' => 'phinxlog',
          'default_environment' => 'development',//默认环境
          'production' => [//生产环境数据库配置
              'adapter' => 'mysql',
              'host' => 'localhost',
              'name' => 'production_db',
              'user' => 'root',
              'pass' => '',
              'port' => '3306',
              'charset' => 'utf8',
          ],
          'development' => [//开发环境数据库配置
              'adapter' => 'mysql',
              'host' => 'localhost',
              'name' => 'development_db',
              'user' => 'root',
              'pass' => '',
              'port' => '3306',
              'charset' => 'utf8',
          ],
          'testing' => [//测试环境数据库配置
              'adapter' => 'mysql',
              'host' => 'localhost',
              'name' => 'testing_db',
              'user' => 'root',
              'pass' => '',
              'port' => '3306',
              'charset' => 'utf8',
          ]
      ],
      'version_order' => 'creation'
  ];
  
  ```

  

- 创建目录db/migrations

  生成的迁移文件将会存储到这个目录，所以目录权限要可读可写

### 迁移

#### 创建迁移

- 生成文件

  使用create命令创建迁移文件

  ```
  ./vendor/bin/phinx create AddTableLgtGood
  ```

  **注意:**迁移文件名需要是驼峰格式,生成的文件格式是`YYYYMMDDHHMMSS_add_table_lgt_good.php`，前面14个日期格式的时间戳+驼峰文件名转成长蛇格式,生成的文件代码如下:

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AddTableLgtGood extends AbstractMigration
  {
      /**
       * Change Method.
       *
       * Write your reversible migrations using this method.
       *
       * More information on writing migrations is available here:
       * https://book.cakephp.org/phinx/0/en/migrations.html#the-change-method
       *
       * Remember to call "create()" or "update()" and NOT "save()" when working
       * with the Table class.
       */
      public function change(): void
      {
  
      }
  }
  ```

- change方法

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AddTableLgtGood extends AbstractMigration
  {
      public function change(): void
      {
          $this->table('lgt_good')
              ->addColumn('lgid','integer')
              ->addColumn('name','string')
              ->addColumn('ctime','datetime')
              ->create();
      }
  }
  ```

  Phinx将自动创建lgt_good表，**注意**如果change方法存在，会忽略存在的up()、down()方法;change方法会自动识别怎么回滚表，使用change方法的前提是使用table的create()、update()方法，下面的这些操作change方法会自动识别如何回滚。

  - 创建表
  - 重命名表
  - 添加列
  - 重命名列
  - 添加索引
  - 添加外键

  如果在change()方法中使用不能回滚的操作时，回滚时将触发IrreversibleMigrationException异常，所以在change()方法中为了防止出现异常报错，可以使用$this->isMigratingUp()方法

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AddTableLgtGood extends AbstractMigration
  {
      public function change(): void
      {
          $table = $this->table('lgt_good');
          $table->addColumn('lgid','integer')
              ->addColumn('name','string')
              ->addColumn('ctime','datetime')
              ->create();
         if($this->isMigratingUp()) {
             $this->table('tbl_users')->insert([['name'=>'hello']])->save();
         }
      }
  }
  ```

- up()方法

  当执行migrate命令时，会自动检测没有执行的文件并执行up()方法，up()方法里面定义数据库的升级

- down()方法

  当执行rollback命令时，会自动检测之前执行的文件的down()方法，down()方法里面定义数据库升级的逆操作。

- init()方法

  执行migrate或者rollback迁移时，在之前up()或者down()方法之前都会执行的方法，常用来初始公共属性，方便up()或者down()方法里面使用

#### 执行sql

execute()方法返回受影响的行数，query()方法返回PDOStatement对象

```php
<?php
declare(strict_types=1);

use Phinx\Migration\AbstractMigration;

final class AddTableLgtGood extends AbstractMigration
{
    public function up(): void
    {
        // execute()
        $count = $this->execute('UPDATE lgt_good set name="hello"'); // returns the number of affected rows
        var_dump($count);
        // query()
        $stmt = $this->query('SELECT * FROM lgt_good'); // returns PDOStatement
        $rows = $stmt->fetchAll(); // returns the result as an array
        print_r($rows);
    }
}
```

#### 查询行

fetchRow()返回单行数据

fetchAll()返回多行数据

```php
<?php
declare(strict_types=1);

use Phinx\Migration\AbstractMigration;

final class AddTableLgtGood extends AbstractMigration
{
    public function up(): void
    {
        // execute()
        $row = $this->fetchRow('SELECT * FROM lgt_good'); // returns array
        var_dump($row);
        // query()
        $list = $this->fetchAll('SELECT * FROM lgt_good'); // returns array list
        print_r($list);
    }

    public function down():void {
        
    }
}
```

#### 写入数据

使用insert()进行写入数据

```php
<?php
declare(strict_types=1);

use Phinx\Migration\AbstractMigration;

final class AddTableLgtGood extends AbstractMigration
{
    public function up(): void
    {
        // inserting only one row
        $singleRow = [
            'lgid'  => 1,
            'name'  => '货品1',
            'ctime' =>  date('Y-m-d H:i:s')
        ];

        $table = $this->table('lgt_good');
        $table->insert($singleRow);
        $table->saveData();

        // inserting multiple rows
        $rows = [
            [
                'lgid'  => 22,
                'name'  => '货品22',
                'ctime' =>  date('Y-m-d H:i:s')
            ],
            [
                'lgid'  => 33,
                'name'  => '货品33',
                'ctime' =>  date('Y-m-d H:i:s')
            ]
        ];

        $table->insert($rows)->save();
    }

    public function down():void {
        $this->execute("DELETE FROM lgt_good");
    }
}
```

**注意:**不要在cange()方法中使用insert进行数据写入,请使用up()、down()进行数据的写入

#### 操作表

- table()方法获取表对象

  ```php
  $this->table("lgt_good");
  ```

- 保存表对象的变化

  - create()方法先创建表然后执行修改的变化
  - update()方法只执行修改，常用在表已经存在，然后对表进行操作的场景
  - save()方法智能的检测，如果表不存在自动执行create()方法，如果表存在则执行save方法

  **注意:**change()方法中只能用create()、update()方法，不能用save()方法，就是因为save()方法智能检测存在则会执行update()方法，不存在则执行create()方法

- 创建表

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AddTableLgtGood extends AbstractMigration
  {
     public function change():void {
         $table = $this->table('lgt_good');
         $table->addColumn('name','string',['length'=>30,'null'=>false])
             ->addColumn('status','integer',['limit'=>11,'default'=>0,'null'=>false])
             ->addIndex('name')
             ->create();
     }
  }
  ```

  addColumn()用来添加列，addIndex()用来添加索引，注意系统自动创建了一个列名为id的主键。

  最终创建的表是:

  ```sql
  CREATE TABLE `lgt_good` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(30) NOT NULL,
    `status` int(11) NOT NULL DEFAULT '0',
    PRIMARY KEY (`id`),
    KEY `name` (`name`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8
  ```

  如果我们想修改主键列名，则可以使用primary_key属性，同时id字段值改为false

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AddTableLgtGood extends AbstractMigration
  {
     public function change():void {
         $table = $this->table('lgt_good',['id'=>false,'primary_key'=>['lgid']]);//更改主键列为lgid
         $table->addColumn('lgid','integer',['length'=>11])
             ->addColumn('name','string',['length'=>30,'null'=>false])
             ->addColumn('status','integer',['limit'=>11,'default'=>0,'null'=>false])
             ->addIndex('name')
             ->create();
     }
  }
  ```

  生成的表为:

  ```sql
  CREATE TABLE `lgt_good` (
    `lgid` int(11) NOT NULL,
    `name` varchar(30) NOT NULL,
    `status` int(11) NOT NULL DEFAULT '0',
    PRIMARY KEY (`lgid`),
    KEY `name` (`name`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8
  ```

  我们可以发现设置primary_key属性的方式生成的主键不是自增的，所以最简单的修改方式如下:

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AddTableLgtGood extends AbstractMigration
  {
     public function change():void {
         $table = $this->table('lgt_good',['id'=>'lgid']);
         $table->addColumn('name','string',['length'=>30,'null'=>false])
             ->addColumn('status','integer',['limit'=>11,'default'=>0,'null'=>false])
             ->addIndex('name')
             ->addColumn('created', 'timestamp', ['default' => 'CURRENT_TIMESTAMP'])
             ->create();
     }
  }
  ```

  生成的表如下:

  ```sql
  CREATE TABLE `lgt_good` (
    `lgid` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(30) NOT NULL,
    `status` int(11) NOT NULL DEFAULT '0',
    `created` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`lgid`),
    KEY `name` (`name`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8
  ```

  lgid字段主键自增了，**注意:**lgid字段不需要addColumn()方法添加了。

  mysql的表创建还支持以下属性:

  - engine 存储引擎,默认*InnoDB*
  - comment 表的备注
  - signed 表主键是否是signed，默认true

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AddTableLgtGood extends AbstractMigration
  {
     public function change():void {
         $table = $this->table('lgt_good',['id'=>'lgid','comment'=>'货品表']);
         $table->addColumn('name','string',['length'=>30,'null'=>false])
             ->addColumn('status','integer',['limit'=>11,'default'=>0,'null'=>false])
             ->addIndex('name')
             ->addColumn('created', 'timestamp', ['default' => 'CURRENT_TIMESTAMP'])
             ->create();
     }
  }
  ```

- 支持的有效列类型
  - biginteger
  - binary
  - boolean
  - date
  - datetime
  - decimal
  - float
  - double
  - integer
  - smallinteger
  - string
  - text
  - time
  - timestamp
  - uuid

- 检查表是否存在

  使用$this->hasTable('表名')方法检查表是否存在

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AddTableLgtGood extends AbstractMigration
  {
     public function change():void {
         if(!$this->hasTable('lgt_good')){
             $table = $this->table('lgt_good',['id'=>'lgid','comment'=>'货品表']);
             $table->addColumn('name','string',['length'=>30,'null'=>false])
                 ->addColumn('status','integer',['limit'=>11,'default'=>0,'null'=>false])
                 ->addIndex('name')
                 ->addColumn('created', 'timestamp', ['default' => 'CURRENT_TIMESTAMP'])
                 ->create();
         }
     }
  }
  ```

- 删除表

  使用drop()方法删除表，drop()方法后通常跟一个save()方法，以保证涉及多个表时phinx智能的进行迁移

  ```php
  $this->table('lt_good')->drop()->save();
  ```

- 重命名表

  ```php
  $this->table('lgt_good')->rename('lgt_goods')->update();
  ```

- 修改表备注

  ```php
  $this->table('lgt_good')->changeComment("货品档案表");
  ```


#### 操作列



- 列可选项

  - 通用可选项

  	- limit 字符串或者整数长度
	
	- length limit的别名
	- default 默认值                            
	- null   是否允许为空，默认false  
	- after   列放到哪个列后面，目前仅支持MySQL
	- comment 列备注 

  - decimal类型选项
    - precision decimal(M,N)中的M
	
   - scale decimal(M,N)中的N                   
   - signed 是否有符号，false无符号，true有符号，仅支持MySQL
   
  - enum类型选项
    - values 可以是逗号隔开的字符串或者是数组 
  - integer、biginteger类型选项

    - identity 是否自增，true或者false
    - signed  是否有符号，仅支持MySQL

  - timestamp类型选项

    - default 默认值，可以使用`CURRENT_TIMESTAMP`
	- update 当行更新是触发变更，值是`CURRENT_TIMESTAMP`，仅支持MySQL

    Phinx提供了快速创建create_at、update_at字段的方法

    ```php
    $this->table('lgt_good')->addTimestamps();//自动创建timestamp类型的create_at、update_at字段
    
    $this->table('lgt_good')->addTimestamps('ctime','utime');//create_at列名改成ctime、update_at列改成utime
    ```

  - boolean类型选项
   - signed 是否有符号，仅支持MySQL

    ```php
    <?php
    declare(strict_types=1);
    
    use Phinx\Migration\AbstractMigration;
    
    final class AddTableLgtGood extends AbstractMigration
    {
       public function change():void {
           $table = $this->table('lgt_good',['id'=>'lgid','comment'=>'货品表']);
           $table->addColumn('name','string',['length'=>30,'null'=>false])
               ->addIndex('name')
               ->addColumn('rate','decimal',['precision'=>5,'scale'=>2,'signed'=>false])
               ->addColumn('created', 'timestamp', ['default' => 'CURRENT_TIMESTAMP'])
               ->addTimestamps('ctime',null)
               ->addColumn('status','boolean')
               ->create();
       }
    }
    ```

    实际创建的是一个tinyint类型

  - 外键选项
 
   - update 设置更新行时触发的事件
   - delete 设置删除行时触发的事件

- getColumns()方法获取表所有列元数据

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AddTableLgtGood extends AbstractMigration
  {
     public function up():void {
         $columns = $this->table('lgt_good')->getColumns();
         print_r($columns);
     }
  
     public function down():void {
  
     }
  }
  
  ```

- getColumn()获取表某一列元数据信息

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AddTableLgtGood extends AbstractMigration
  {
     public function up():void {
         $column = $this->table('lgt_good')->getColumn('status');
         print_r($column);
     }
  
     public function down():void {
  
     }
  }
  ```

- hasColumn()检测列是否存在

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AddTableLgtGood extends AbstractMigration
  {
     public function up():void {
         $ret = $this->table('lgt_good')->hasColumn('status');
         print_r($ret);
     }
  
     public function down():void {
  
     }
  }
  ```

- renameColumn()重命名列名

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AlterLgtGood extends AbstractMigration
  {
     public function up():void {
         $table = $this->table('lgt_good');
         $table->renameColumn('status','lstatus')->update();
     }
  
     public function down():void {
         $table = $this->table('lgt_good');
         $table->renameColumn('lstatus','status')->update();
     }
  }
  ```

- removeColumn()删除列

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AlterLgtGood extends AbstractMigration
  {
     public function up():void {
         $table = $this->table('lgt_good');
         $table->removeColumn('status')
             ->update();
     }
  
     public function down():void {
         $table = $this->table('lgt_good');
         $table->addColumn('status','boolean',['default'=>1])
             ->update();
     }
  }
  ```

- changeColumn()修改列属性

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AlterLgtGood extends AbstractMigration
  {
     public function up():void {
         $table = $this->table('lgt_good');
         $table->changeColumn('status','integer')
             ->update();
     }
  
     public function down():void {
         $table = $this->table('lgt_good');
         $table->changeColumn('status','boolean',['default'=>1])
             ->update();
     }
  }
  ```

  

#### 操作索引

- addIndex()添加索引

- removeIndex()删除索引

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AlterLgtGood extends AbstractMigration
  {
     public function up():void {
         $table = $this->table('lgt_good');
         $table->addIndex('name')
             ->update();
     }
  
     public function down():void {
         $table = $this->table('lgt_good');
         $table->removeIndex('name')
             ->update();
     }
  }
  ```

  addIndex()默认创建普通索引，创建唯一索引需要加参数unique

  ```php
  <?php
  declare(strict_types=1);
  
  use Phinx\Migration\AbstractMigration;
  
  final class AlterLgtGood extends AbstractMigration
  {
     public function up():void {
         $table = $this->table('lgt_good');
         $table->addIndex('name',[
             'unique' =>  true,
             'name'   =>  'name_index'
         ])
             ->update();
     }
  
     public function down():void {
         $table = $this->table('lgt_good');
         $table->removeIndex('name')
             ->update();
     }
  }
  ```

  上面创建了一个索引名称是name_index的唯一索引。

  也可以创建组合索引，如下

  ```php
  $table->->addIndex(['email','username'])
  ```

  也可以给字段的指定长度创建索引，如下:

  ```php
  $table->->addIndex('name',['limit'=>10])
  ```

  

#### 使用查询生成器

查询生成器用来操作负载的SELECT、UPDATE、DELETE、INSERT,

使用如下代码获取生成器对象:

```php
$builder = $this->getQueryBuilder();
```

- 查询字段

  ```php
  $builder->select(['id', 'title', 'body']);
  
  // Results in SELECT id AS pk, title AS aliased_title, body ...
  $builder->select(['pk' => 'id', 'aliased_title' => 'title', 'body']);
  
  // Use a closure
  $builder->select(function ($builder) {
      return ['id', 'title', 'body'];
  });
  ```

- where条件

  ```php
  // WHERE id = 1
  $builder->where(['id' => 1]);
  
  // WHERE id > 1
  $builder->where(['id >' => 1]);
  
  $builder->where(['id >' => 1])->andWhere(['title' => 'My Title']);
  
  // Equivalent to
  $builder->where(['id >' => 1, 'title' => 'My title']);
  
  // WHERE id > 1 OR title = 'My title'
  $builder->where(['OR' => ['id >' => 1, 'title' => 'My title']]);
  ```

  更复杂的查询

  ```php
  // Coditions are tied together with AND by default
  $builder
      ->select('*')
      ->from('articles')
      ->where(function ($exp) {
          return $exp
              ->eq('author_id', 2)
              ->eq('published', true)
              ->notEq('spam', true)
              ->gt('view_count', 10);
      });
  ```

  等价于下面的sql:

  ```sql
  SELECT * FROM articles
  WHERE
      author_id = 2
      AND published = 1
      AND spam != 1
      AND view_count > 10
  ```

  组合查询

  ```php
  $builder
      ->select('*')
      ->from('articles')
      ->where(function ($exp) {
          $orConditions = $exp->or_(['author_id' => 2])
              ->eq('author_id', 5);
          return $exp
              ->not($orConditions)
              ->lte('view_count', 10);
      });
  ```

  生成的sql如下:

  ```php
  SELECT *
  FROM articles
  WHERE
      NOT (author_id = 2 OR author_id = 5)
      AND view_count <= 10
  ```

  支持where的可选方法还有下列一些:

  - eq()
  - notEq()
  - like()
  - notLike()
  - in()
  - notIn()
  - gt()
  - gte()
  - lt()
  - lte()
  - isNull()
  - isNotNull()

  使用MySQL内置方法的方式:

  ```php
  // Results in SELECT COUNT(*) count FROM ...
  $builder->select(['count' => $builder->func()->count('*')]);
  ```

- 获取查询结果

  ```php
  // Iterate the query
  foreach ($builder as $row) {
      echo $row['title'];
  }
  
  // Get the statement and fetch all results
  $results = $builder->execute()->fetchAll('assoc');
  ```

- 创建数据

  ```php
  $builder = $this->getQueryBuilder();
  $builder
      ->insert(['first_name', 'last_name'])
      ->into('users')
      ->values(['first_name' => 'Steve', 'last_name' => 'Jobs'])
      ->values(['first_name' => 'Jon', 'last_name' => 'Snow'])
      ->execute()
  ```

- 更新数据

  ```php
  $builder = $this->getQueryBuilder();
  $builder
      ->update('users')
      ->set('fname', 'Snow')
      ->where(['fname' => 'Jon'])
      ->execute()
  ```

- 删除数据

  ```php
  $builder = $this->getQueryBuilder();
  $builder
      ->delete('users')
      ->where(['accepted_gdpr' => false])
      ->execute()
  ```

### 常用命令

- 创建迁移文件

  ```shell
  ./vendor/bin/phinx create MyMigrate
  ```

- 初始化，生成Phinx的配置文件

  ```php
  ./vendor/bin/phinx init  //默认当前目录生成phinx.php
  ```

  ```php
  ./vendor/bin/phinx init migration.php //指定生成的文件名
  ```

  ```php
  ./vendor/bin/phinx init --format=yml //指定生成的文件格式，可以指定php、json、yml
  ```

- 执行迁移

  ```shell
  ./vendor/bin/phinx migrate -e development  
  ```

  -e指定数据库环境，不指定-e,默认使用配置文件中配置的默认环境

- 执行回滚

  ```php
  phinx rollback -e development
  ```

### 配置

- 指定迁移文件生成的位置

  ```php
  paths:
      migrations: /your/full/path
  ```

- 指定迁移数据表名

  ```php
  environments:
      default_migration_table: migrations
  ```

- 配置默认环境

  ```php
  environments:
      default_environment: development
  ```

- 表前缀、后缀

  ```php
  environments:
      development:
          table_prefix: dev_
          table_suffix: _v1
  ```

  

### 更多

Phinx是一个强大的工具，还有很多功能没能及时进行说明，可以参考[Phinx官网](https://phinx.org/)进行学习

