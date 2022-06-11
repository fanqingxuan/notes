### 其它
### 测试mysql版本5.7.29

##### 创建json字段表

```sql
create table test (
	id int(11) NOT NULL PRIMARY KEY AUTO_INCREMENT,
	data json
) charset=utf8;
```

写入字段数据,插入满足json格式的字符串，不符合json格式的字符串插入时会报错

```sql
 insert into test (data) values ('dddd');  -- 非json格式会报错
 insert into test (data) values ('{"name":"范晓杰","age":34}'); 
 insert into test (data) values ('[11,22,33]');
```

##### 更新字段数据

```sql
update test set data = '{"name":"json"}' where id=1;
```

##### json函数

- json_array 构建json列表数据

  ```sql
  select json_array(11,22,33,44);-- 查询
  insert into test(data) values (json_array(33,44,55));-- 插入
  ```

- json_object构建son对象数据

  ```sql
  select json_object("name","json","age",12); -- json_object 构建对象数据，key value，**值必须是偶数个**，构成key,value，否则会报错
  insert into test(data) values (json_object("name","json","age",12))
  select json_object(33,44,55) -- 奇数个报错
  ```

**注意value一定要是偶数个，否则会报错 **

- json_merge_preserve合并多个json数据

  ```sql
  select json_merge_preserve('[11,22]','{"name":4}','[44]','[]'); -- json_merge_preserve 合并多个json格式数据
  
  insert into test(data) values (json_merge_preserve('[11,22]','{"name":4}','[44]','[]'));
  
  insert into test(data) values (json_merge_preserve(json_array(11,22),json_array(11,23))); -- 综合各种函数
  ```

- json_keys返回对象的所有key

  ```sql
  select json_keys(data) from test;
  ```

- JSON_REPLACE 替换值（只替换已经存在的旧值，不存在则不写入）

  ```sql
  select json_replace('{"name":"json"}','$.name',"范晓杰"); -- name替换成了范晓杰
  select json_replace('{"name":"json"}','$.age',"23"); -- 没有写入age属性
  update test set data = json_replace('{"name":"json"}','$.age',"23") where id=1;
  ```

- JSON_SET 设置值（替换旧值或插入不存在的新值）

  ```sql
  select json_set('{"name":"json"}','$.name',"范晓杰"); -- name替换成了范晓杰
  select json_set('{"name":"json"}','$.age',"23"); -- 写入age属性
  ```

- JSON_INSERT 插入值（插入新值，不替换已经存在的旧值）

  ```SQL
  select json_insert('{"name":"json"}','$.name',"范晓杰"); -- name没有替换，仍然是json
  select json_insert('{"name":"json"}','$.age',"23"); -- 写入了age属性
  ```

- JSON_REMOVE 删除JSON数据，删除指定值后的JSON文档

  ```sql
  select json_remove('{"name":"json"}','$.name'); -- 删除了name属性
  ```

**json还提供了许多其它函数，可自行查询网络资料，注意看当前使用的mysql版本是否支持该函数，一定要实践**

##### json字段查询

​    mysql5.7版本提供了两种方式获取json字段的单值

  1. json_extract解析json的值

  2. column->path的形式

  3. 用法

     - .keyName 获取JSON对象中键名为keyName的值

       ```sql
       SELECT json_extract(data,'$.name') from test; -- 获取data字段中key为name的value
       SEKECT data->'$.name' from test;--等价于select json_extract(data,'$.name') from test
       SELECT data->>'$.name' from test; --等价于select json_unquote(json_extract(data,'$.name')) from test;
       ```

     - 对于不合法的键名（如有空格），在路径引用中必须用双引号"将键名括起来

       ```sql
       select json_extract(data,'$."address info"') from test;
       select data->'$."address info"' from test;
       ```

     - [index]：JSON数组中索引为index的值，JSON数组的索引同样从0开始

       ```sql
       select json_extract(data,'$[0]') from test;
       select data->'$[0]' from test;
       ```

     - [ index1 to index2]：~~JSON数组中从index1到index2的值的集合,可能跟版本有关系，mysql5.7.29版本验证没有通过~~

       ```sql
       select data->'$[0 to 2]' from test;
       ```

     - .*:JSON对象中的所有value,**注意是value，不获取key**

       ```sql
       select json_extract(data,'$.*') from test;
       select data->'$.*' from test;
       ```

     - [*] :JSON数组中的所有值

       ```sql
       select data->'$[*]' from test;
       select json_extract(data,'$[*]') from test;
       ```

     - 查询不存在的属性返回结果为NULL

     - 支持访问嵌套属性

       ```sql
       select data->'$.list[0].age' from test;
       ```

**注意column->>key方式可以去掉value的引号，同json_unquote(json_extract(column,'$.key'))**

##### json字段where条件过滤

- where后面直接使用json_extract或者column->key格式

  ```sql
  select * from test where json_extract(data,'$.age')>10;  -- where后面使用json_extract 走全表扫描，不会使用索引
  select * from test where data->'$.age'=10;
  ```

- 虚拟列方式

  ```sql
  ALTER TABLE test  ADD name_virtual  varchar (32) GENERATED ALWAYS  AS (json_extract(data,  '$.name' )) VIRTUAL;
  
  select * from test where name_virtual='"json"'; -- 像正常字段一样访问虚拟列
  select name_virtual from test;
  ```

  **注意:**   

  1. 插入数据时不可以向虚拟列插入数值,也不可以更新虚拟列值，否则会报错，只需要更新或写入json字段值，mysql会自动映射虚拟列值

  2. 创建虚拟列是varchar类型，并且where条件是虚拟列

     - 若创建表时就创建了虚拟列，则查询条件值需要加双引号

       ```sql
       select * from test where name_virtual='"json"';
       ```

     - 若表中已经有数据，之后才创建了虚拟列，虚拟列映射的值可能有数值类型，也可能有字符串类型，where条件带双引号和不带双引号可能都不合适,  建议创建json字段时根据业务场景就要创建好虚拟列，或者一定要保证某key数据类型的一致性

  3. 使用 column->key和json_extract方式不需要加双引号，mysql内部会自动过滤

     ```sql
     select * from test where data->'$.num'='34';
     ```

   - 对虚拟列创建索引

        ```sql
        CREATE INDEX name_virtual_index  ON test(name_virtual);
        ```

#####  48环境坑 

- 执行任何sql报错
           Expression #1 of ORDER BY clause is not in GROUP BY clause and contains nonaggregated column 'information_schema.PROFILING.SEQ' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
           原因:sql _mode中only _full _group _by不兼容
           暂时解决方式:SET sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));
           根治方式:my.ini 配置sql_mode
- 创建包含json字段的表，查询不到json字段
          create table test(
              id int(11) NOT NULL PRIMARY KEY AUTO_INCREMENT,
              data json,
              age int(11)
          );
          select * from test;   -- 只查询到了id,age列
        原因:**可能是navicat客户端的问题，用mysql命令行，可以正常展示json字段列**     

##### php代码测试json字段	

```php
//mysqli方式
 $conn = mysqli_connect("hostname","root","a12345","demo");

 mysqli_query($conn, "set names utf8");

 $ret = mysqli_query($conn, "select * from test");

 $row = mysqli_fetch_assoc($ret);
 var_dump($row);

//pdo方式
 $pdo = new PDO("mysql:host=hostname;dbname=demo","root","a12345");
 $pdo->query("set names utf8");
 $result = $pdo->query("SELECT * FROM test"); 
```

- 结论
  1. php不能将json字段自动解析成php数组，mysql的json数据字段返回的结果是json字符串，如果需要使用,则需要php json_decode解析成数组
  2. **查询json的varchar虚拟列，若json字段该key存储的是字符串，返回的结果是带有双引号的值，如"范晓杰",而不是期望的:范晓杰，若json字段该key存储的是整形，则返回的结果是常规字符串,需要特别注意**
     - 解决方式:
       - 结果在php层面使用json_decode解析
       - 使用mysql函数JSON_UNQUOTE去双引号，例如:select JSON_UNQUOTE(data->'$.name') from test;

##### 进阶技巧分享

​	如果你通过上面的测试，以及结合php测试，可以发现有如下的几个不方便的地方

- 查询json某字符串key返回的是带有双引号的值，如"value"
- where条件之后使用data->'$.name'='"xxxx"'的形式，或者json_extract(data,'$.name')='xx'的形式会全表扫描
- where条件包含虚拟列过滤时，需要特别注意添加"",例如name_virtual='"json"',代码层面可能多很多双引号，必要时需要使用\去转义，也很容易忘记添加双引号造成不必要的bug
- 返回的字符串虚拟列也带有双引号，如"json",业务代码需要json_decode特殊处理

基于以上问题有如下解决方式

```sql
ALTER TABLE test  ADD name_virtual_1  varchar (32) GENERATED ALWAYS  AS (json_unquote(json_extract(data,  '$.name' ))) VIRTUAL; -- 不带双引号的value,创建虚拟列
alter table test add index name_virtual_index_1(name_virtual_1); --虚拟列加索引
```

通过以上sql解决了问题

- 通过虚拟列查询，使用了索引，可以用explain查看
- where条件不需要"json"了，直接使用json，跟其它字符串查询一致，不需要使用select * from test where name_virtual='"json"'，直接使用select * from test where name_virtual='json';
- 返回的name_virtual_1字段不再包含""，跟字符串一样, 不再需要业务层json_decode

#####  其它注意问题

- 创建虚拟列要考虑历史数据的字段类型，json字段里面不限制字段类型，比如表里已经存在数据{"name":"json","age":"范晓杰"}，执行如下sql

  ```sql
  ALTER TABLE test  ADD age  int(11) GENERATED ALWAYS  AS (json_unquote(json_extract(data,  '$.age' )));
  ```

  再写入数据则会报错，即使写入的age字段是int类型也会报错，所以要写入正确的数据，或者根据业务场景在创建json字段时及时创建虚拟列

- 不要使用json_set(data,'$.name',NULL)试图删除name列，mysql会存储为null字符串，php查询结果是"null"而不是NULL，要使用json_remove