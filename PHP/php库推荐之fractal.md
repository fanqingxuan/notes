### PHP
### 前言

今天推荐的这个库叫`league/fractal`，用途是在源数据和响应结果之间提供一种转换方式。

### 优点

- Fractal 为复杂的数据输出提供了一个表示和转换层，为表示层提供固定的数据结构，提高代码的复杂度。

- 当表示层或者api接口开发需要部分字段时，会大量使用循环，去组装响应的结果数据，使用Fractal可以避免代码中循环。

- Fractal将源数据转换成固定的数据结构，这样源数据有字段增删时，不会影响表示层的数据结构。
- 有时候返回表示层的数据可能来源于多个数据源，或者嵌套关系，使用Fractal可以拆分逻辑，降低代码复杂度

### 安装方式

```shell
composer require league/fractal
```

### 代码示例

```php
<?php

require 'vendor/autoload.php';

use League\Fractal\Manager;
use League\Fractal\Resource\Collection;

// Create a top level instance somewhere
$fractal = new Manager();

//原始数据
$userlist = [
	[
		'user_id'   => '1',
		'name'      => 'Json',
		'email'     => 'fanqingxuan@163.com',
		'age'       => 32,
	],
	[
		'user_id'   => '2',
		'name'      => '范先生',
		'email'     => 'json@163.com',
		'age'       => 29,
	]
];

$resource = new Collection($userlist, function(array $user) {
    return [
        'id'            => (int) $user['user_id'],
        'username'      => $user['name'],
        'email'         => $user['email'],
        'other'         => [
        	'age'   => $user['age'],
        ],
        'url'           => "user/".$user['user_id']
    ];
});

$array = $fractal->createData($resource)->toArray();
//print_r($array);
// Turn all of that into a JSON string
echo $fractal->createData($resource)->toJson();
//输出的是:{"data":[{"id":1,"username":"Json","email":"fanqingxuan@163.com","other":{"age":
32},"url":"user\/1"},{"id":2,"username":"\u8303\u5148\u751f","email":"json@163.c
om","other":{"age":29},"url":"user\/2"}]}
```

我们可以看到输出的结构我们进行了重新规范，可能我们看着代码有些多，不过可以通过使用Transformers类进行简化，简化用法我们稍后再讲。

### 概念讲解

- 资源Resources

  上面的例子中使用了new Collection，这在Fractal里面叫Resources，Resources主要作用是对不同的结构数据进行对象化处理，然后通过使用Transformer或者会掉函数输出固定的结构。Fractal包含两种常用的Resources，就是Item和Collection。

  - Collection对集合或者List的结构进行对象化处理。

    上面的源数据userlist是个list所以用Collection

  - Item用在对一维数组或者单挑记录进行对象化。

    ```php
    //原始数据
    $userInfo = [
        'user_id'   => '2',
        'name'      => '范先生',
        'email'     => 'json@163.com',
        'age'       => 29,
    ];
    
    $resource = new Item($userInfo, function(array $user) {
        return [
            'id'            => (int) $user['user_id'],
            'username'      => $user['name'],
            'email'         => $user['email'],
            'other'         => [
            	'age'   => $user['age'],
            ],
            'url'           => "user/".$user['user_id']
        ];
    });
    
    $array = $fractal->createData($resource)->toArray();
    print_r($array);
    
    echo $fractal->createData($resource)->toJson();
    ```

- Serializers

  Serializer 是以某种方式构建你的转换后的数据。Fractal默认提供了几种不同的Serializer, 这些 Serializers 之间的大多数差异是，如何定义数据的命名空间。Serializer 让我们在 Transformers 上可进行最小的修改，然后在不同的输出格式之间切换。设计Serializer方式如下:

  ```php
  $fractal->setSerializer(new League\Fractal\Serializer\ArraySerializer);
  ```

  - DataArraySerializer

    这是 Fractal 中的默认Serializer，该序列化并不符合每个人的口味，因为它给输出添加了一个 'data' 的命名空间。比如上面的结果输出,我们可以看到将结构放到了data域下。

    ```php
    {"data":{"id":2,"username":"\u8303\u5148\u751f","email":"json@163.com","other":{
    "age":29},"url":"user\/2"}}
    ```

    使用 'data' 命名空间，非常适合标准化开发，比如api开发规范通常是把数据放到data下给客户端使用，此外使用data命名空间也可以防止命名冲突，还可以引入其他非data域下的资源。

  - ArraySerializer

    如果确实想移除 'data' 命名空间，这可以通过使用 ArraySerializer 来实现。除了 移除了'data'命名空间之外，跟DataArraySerializer大体相同。这里值得注意的是，Collections 不会移除 'data' 命名空间，只有Item会移除。我想Collection不移除可能是考虑到方式跟元数据(meta data)冲突，另外如果移除data命名空间，返回结构可能类似如下,客户端难以获取数据

    ```jso
    {"0":{"id":1,"username":"Json","email":"fanqingxuan@163.com","other":{"age":32},
    "url":"user\/1"},"1":{"id":2,"username":"\u8303\u5148\u751f","email":"json@163.c
    om","other":{"age":29},"url":"user\/2"},"meta":[]}
    ```

  - 自定义 Serializers

    我们可以继承SerializerAbstract抽象类来定义自己的Serializers。

    ```php
    $manager->setSerializer(new CustomSerializer());//使用自定义Serializers，进行数据序列化
    ```

- Transformer

  Transformer定义数据的结构，有回调和使用Transformer类两种方式

  - 回调

    在某些情况下，使用这种方式是比较便捷的。上面我们使用了回调，类似下面的代码

  ```php
  new Collection($userlist, function(array $user) {
      return [
          'id'            => (int) $user['user_id'],
          'username'      => $user['name'],
          'email'         => $user['email'],
          'other'         => [
          	'age'   => $user['age'],
          ],
          'url'           => "user/".$user['user_id']
      ];
  });
  ```

  - 定义Transformer类

    大多数数据需要在多个位置进行多次转换, 因此创建 Transformer类的方式，可以减少代码重复，便于代码复用，同时也可以简化业务层代码量。

    Transformer类必须继承 League\Fractal\TransformerAbstract，并且至少实现方法 transform() ;

    我们将上面的代码改成Transformer类方式,Transformer类大概如下:

    ```php
    class UserTransformer extends League\Fractal\TransformerAbstract
    {
        public function transform(array $userInfo)
        {
            return [
                'id'            => (int) $userInfo['user_id'],
                'username'      => $userInfo['name'],
                'email'         => $userInfo['email'],
                'other'         => [
                    'age'   => $userInfo['age'],
                ],
                'url'           => "user/".$userInfo['user_id']
            ];
        }
        
    }
    ```

    我们可以看到将回调函数的数据结构定义封装到了transform方法里面，一旦 Transformer 类被定义，就可以将其作为实例，传递给 resource 构造函数，如下代码:

    ```php
    $resource = new Collection($userlist, new UserTransformer);
    $array = $fractal->createData($resource)->toArray();
    ```

### 高级玩法

通过上面的概念讲解，其实已经把Fractal的基本使用方式说了一遍，那我们下面讲下引入数据的用法，比如我们返回的结构可能是用户list，然后用户下面又有文章列表的情况，我们也想规范化输出。

常规的方法，在回调或者transform方法里面进行循环结构化，如下使用方式:

```php
<?php

require 'vendor/autoload.php';

use League\Fractal\Manager;
use League\Fractal\Resource\Collection;

// Create a top level instance somewhere
$fractal = new Manager();

//原始数据
$userlist = [
	[
		'user_id'   => '1',
		'name'      => 'Json',
		'email'     => 'fanqingxuan@163.com',
		'age'       => 32,
        'list'      =>  [
            [
                'id'    =>  1,
                'title' =>  '文章1',
                'text'  =>  '文章1内容',
                'status'=>  1
            ],
            [
                'id'    =>  2,
                'title' =>  '文章2',
                'text'  =>  '文章2内容',
                'status'=>  0
            ]
        ]
	],
	[
		'user_id'   => '2',
		'name'      => '范先生',
		'email'     => 'json@163.com',
		'age'       => 29,
        'list'      =>  [
            [
                'id'    =>  3,
                'title' =>  '文章3',
                'text'  =>  '文章3内容',
                'status'=>  1
            ],
            [
                'id'    =>  4,
                'title' =>  '文章4',
                'text'  =>  '文章4内容',
                'status'=>  1
            ]
        ]
	]
];

class UserTransformer extends League\Fractal\TransformerAbstract
{
    public function transform(array $userInfo)
    {
        $info = [
            'id'            => (int) $userInfo['user_id'],
            'username'      => $userInfo['name'],
            'email'         => $userInfo['email'],
        ];
        foreach($userInfo['list'] as $post) {
            $info['posts'][] = [
                'post_id'   =>  $post['id'],
                'title'     =>  $post['title']
            ];
        }
        return $info;
    }
    
}

$resource = new Collection($userlist, new UserTransformer);

$array = $fractal->createData($resource)->toArray();

print_r($array);
```

上面的代码不是不可以，但是难免感觉有些臃肿，达不到复用效果。Fractal为我们提供了更好的方式，避免显示for循环，使用include方式，如下代码:

```php
<?php

require 'vendor/autoload.php';

use League\Fractal\Manager;
use League\Fractal\Resource\Collection;

// Create a top level instance somewhere
$fractal = new Manager();

//原始数据
$userlist = [
	[
		'user_id'   => '1',
		'name'      => 'Json',
		'email'     => 'fanqingxuan@163.com',
		'age'       => 32,
        'list'      =>  [
            [
                'id'    =>  1,
                'title' =>  '文章1',
                'text'  =>  '文章1内容',
                'status'=>  1
            ],
            [
                'id'    =>  2,
                'title' =>  '文章2',
                'text'  =>  '文章2内容',
                'status'=>  0
            ]
        ]
	],
	[
		'user_id'   => '2',
		'name'      => '范先生',
		'email'     => 'json@163.com',
		'age'       => 29,
        'list'      =>  [
            [
                'id'    =>  3,
                'title' =>  '文章3',
                'text'  =>  '文章3内容',
                'status'=>  1
            ],
            [
                'id'    =>  4,
                'title' =>  '文章4',
                'text'  =>  '文章4内容',
                'status'=>  1
            ]
        ]
	]
];

class PostTransformer extends League\Fractal\TransformerAbstract
{
                    
    public function transform(array $post)
    {
        return [
            'post_id'   =>  $post['id'],
             'title'    =>  $post['title']
        ];
    }
    
}

class UserTransformer extends League\Fractal\TransformerAbstract
{
    protected $defaultIncludes = ["posts"];
                    
    public function transform(array $userInfo)
    {
        return [
            'id'            => (int) $userInfo['user_id'],
            'username'      => $userInfo['name'],
            'email'         => $userInfo['email'],
        ];
    }
    
    public function includePosts($userInfo)
    {
        $posts = $userInfo['list'];
        return $this->collection($posts,new PostTransformer);
    }
    
}

$resource = new Collection($userlist, new UserTransformer);

$array = $fractal->createData($resource)->toJSON();

print_r($array);
```

我们可以看到将post相关的结构，抽离成了一个PostTransformer类，这样不仅简化代码，同时可以复用。另外在

UserTransformer定义了一个includePosts方法，这里注意注意的是:

- includePosts是一种magic写法，方法规则是includeXXX，引入数据的资源命名空间就是xxx，例如includePost则引入数据的顶级命名空间就是posts，如下:

  ```php
  {"data":[{"id":1,"username":"Json","email":"fanqingxuan@163.com","posts":{"data":[{"post_id":1,"title":"\u6587\u7ae01"},{"post_id":2,"title":"\u6587\u7ae02"}]}},{"id":2,"username":"\u8303\u514
  8\u751f","email":"json@163.com","posts":{"data":[{"post_id":3,"title":"\u6587\u7ae03"},{"post_id":4,"title":"\u6587\u7ae04"}]}}]}
  ```

  

- 引入数据的Transformer需要定义$defaultIncludes属性，这个属性是一个数组。定义了includeXxx方法，若该属性中添加了xxx，则生成的最终结构就会包含顶级命名空间是xxx的资源。如上代码中的

  ```php
  protected $defaultIncludes = ['posts'];
  ```

  

- $defaultIncludes是一个默认include的方式，若定义了includeXxx方法，返回结构就会包含默认资源的结构。Fractal提供了可选的方式，即可以动态设置是否返回引入数据资源，这个需要在Transformer的继承类里面使用$availableIncludes定义可用的资源，然后使用parseIncludes方法解析需要展示的资源，举例如下:

  ```php
  class UserTransformer extends League\Fractal\TransformerAbstract
  {
      protected $availableIncludes = ['posts'];//可引入的资源
                      
      public function transform(array $userInfo)
      {
          return [
              'id'            => (int) $userInfo['user_id'],
              'username'      => $userInfo['name'],
              'email'         => $userInfo['email'],
          ];
      }
      
      public function includePosts($userInfo)//可用资源的方法includeXxx
      {
          $posts = $userInfo['list'];
          return $this->collection($posts,new PostTransformer);
      }
      
  }
  
  $resource = new Collection($userlist, new UserTransformer);
  
  $array = $fractal->createData($resource)->toJSON();
  print_r($array);
  $fractal->parseIncludes('posts');//解析要引入的资源
  $array = $fractal->createData($resource)->toJSON();
  print_r($array);
  ```

  输出结果如下,没有使用parseIncludes()方法时,即使定义了$availableIncludes属性，也不会不包含引用的资源。

  ```php
  {"data":[{"id":1,"username":"Json","email":"fanqingxuan@163.com"},{"id":2,"username":"\u8303\u5148\u751f","email":"json@163.com"}]}
  ```

  ```php
  {"data":[{"id":1,"username":"Json","email":"fanqingxuan@163.com","posts":{"data":[{"post_id":1,"title":"\u6587\u7ae01"},{"post_id":2,"title":"\u6587\u7ae02"}]}},{"id":2,"username":"\u8303\u514
  8\u751f","email":"json@163.com","posts":{"data":[{"post_id":3,"title":"\u6587\u7ae03"},{"post_id":4,"title":"\u6587\u7ae04"}]}}]}
  ```

  **注意:parseIncludes方法支持引入多个资源,用逗号分开,也支持数组**

  ```php
  $fractal->parseIncludes('posts,order');
  $fractal->parseIncludes(['posts','order']);
  ```

### 代码封装

上面我们就把Fractal的用法讲完了，为了更好的熟悉，以及灵活应用，笔者特意做了一个封装的demo，大家可以尝试用用

- 安装

  ```php
  composer require fanqingxuan/presenter
  ```

- 使用

  ```php
  require 'vendor/autoload.php';
  
  use Json\TransformerAbstract;
  use Json\Presenter;
  
  class BookTransformer extends TransformerAbstract
  {
      public function transform($book)
      {
          return [
              'id'      => (int) $book['id'],
              'title'   => $book['title'],
              'year'    => (int) $book['yr'],
          ];
      }
  }
  
  $presenter = new Presenter();
  
  //模拟源数据
  $books =[
          'id' => '1',
          'title' => 'Hogfather',
          'yr' => '1998',
          'author_name' => 'Philip K Dick',
          'author_email' => 'philip@example.org',
      ];
  $data = $presenter->transform($books,new BookTransformer(),false);//不包含引用资源,输出单记录结构
  ```

  输出结果

  ```shell
  Array
  (
      [id] => 1
      [title] => Hogfather
      [year] => 1998
  )
  ```

- 说明

  - 输出单记录结构

    ```php
    //模拟源数据
    $books =[
            'id' => '1',
            'title' => 'Hogfather',
            'yr' => '1998',
            'author_name' => 'Philip K Dick',
            'author_email' => 'philip@example.org',
        ];
    $data = $presenter->transform($books,new BookTransformer(),false);//不包含引用资源,输出单 ```
    **注意transform第三个参数是false，表示数据源是单记录结构**
  - 输出单记录结构，且结构引入其他资源
    - 方式一
	```php
      //定义transformer
      class LinkTransformer extends TransformerAbstract
      {
          public function transform($book)
          {
              return [
                  'rel' => 'self',
                   'uri' => '/books/'.$book['id'],
              ];
          }
      }   
      class BookTransformer extends TransformerAbstract
      {
          /**
           * List of resources possible to include
           *
           * @var array
           */
          protected $availableIncludes = [
              'links'
          ];
		  
          public function transform($book)
          {
              return [
                  'id'      => (int) $book['id'],
                  'title'   => $book['title'],
                  'year'    => (int) $book['yr'],
              ];
          }
      
          public function includeLinks($book)
          {
              return $this->item($book,new LinkTransformer);
          }
      }
      
      $presenter = new Presenter();
      
      //模拟源数据
      $books =[
              'id' => '1',
              'title' => 'Hogfather',
              'yr' => '1998',
              'author_name' => 'Philip K Dick',
              'author_email' => 'philip@example.org',
          ];
      $data = $presenter->transform($books,new BookTransformer(['links']),false);//包含引用记录,输出单记录结构
      print_r($data);
```

      **Transformer类的构造函数传引用的资源，可以是数组或者字符串** 
	- 方式二
      通过Transformer的setAvailableIncludes()方法设置引用的资源，参考可以是数组或者字符串，字符串用逗号区分多个资源。
	```php
      //定义transformer
      class LinkTransformer extends TransformerAbstract
      {
          public function transform($book)
          {
              return [
                  'rel' => 'self',
                   'uri' => '/books/'.$book['id'],
              ];
          }
      }

      class BookTransformer extends TransformerAbstract
      {
          /**
           * List of resources possible to include
           *
           * @var array
           */
          protected $availableIncludes = [
              'links'
          ];

          public function transform($book)
          {
              return [
                  'id'      => (int) $book['id'],
                  'title'   => $book['title'],
                  'year'    => (int) $book['yr'],
              ];
          }
      
          public function includeLinks($book)
          {
              return $this->item($book,new LinkTransformer);
          }
      }
      
      $presenter = new Presenter();
      
      //模拟源数据
      $books =[
              'id' => '1',
              'title' => 'Hogfather',
              'yr' => '1998',
              'author_name' => 'Philip K Dick',
              'author_email' => 'philip@example.org',
          ];
      
      //包含引用记录,输出单记录结构的另一种方式
      $bookTransformer = new BookTransformer;
      $bookTransformer->setAvailableIncludes('links');
      $data = $presenter->transform($books,$bookTransformer,false);//包含引用记录,输出单记录结构
      print_r($data);
	```
  - 输出集合资源

    ```php
    require 'vendor/autoload.php';
    
    use Json\TransformerAbstract;
    use Json\Presenter;
    
    class BookTransformer extends TransformerAbstract
    {
        public function transform($book)
        {
            return [
                'id'      => (int) $book['id'],
                'title'   => $book['title'],
                'year'    => (int) $book['yr'],
            ];
        }
    }
    
    $presenter = new Presenter();
    
    //原始数据
    $userlist = [
    	[
    		'user_id'   => '1',
    		'name'      => 'Json',
    		'email'     => 'fanqingxuan@163.com',
    		'age'       => 32,
            'list'      =>  [
                [
                    'id'    =>  1,
                    'title' =>  '文章1',
                    'text'  =>  '文章1内容',
                    'status'=>  1
                ],
                [
                    'id'    =>  2,
                    'title' =>  '文章2',
                    'text'  =>  '文章2内容',
                    'status'=>  0
                ]
            ]
    	],
    	[
    		'user_id'   => '2',
    		'name'      => '范先生',
    		'email'     => 'json@163.com',
    		'age'       => 29,
            'list'      =>  [
                [
                    'id'    =>  3,
                    'title' =>  '文章3',
                    'text'  =>  '文章3内容',
                    'status'=>  1
                ],
                [
                    'id'    =>  4,
                    'title' =>  '文章4',
                    'text'  =>  '文章4内容',
                    'status'=>  1
                ]
            ]
    	]
    ];
    
    class PostTransformer extends TransformerAbstract
    {
                        
        public function transform(array $post)
        {
            return [
                'post_id'   =>  $post['id'],
                 'title'    =>  $post['title']
            ];
        }
        
    }
    
    class UserTransformer extends TransformerAbstract
    {
        protected $availableIncludes = ['posts'];
                        
        public function transform(array $userInfo)
        {
            return [
                'id'            => (int) $userInfo['user_id'],
                'username'      => $userInfo['name'],
                'email'         => $userInfo['email'],
            ];
        }
        
        public function includePosts($userInfo)
        {
            $posts = $userInfo['list'];
            return $this->collection($posts,new PostTransformer);
        }
        
    }
    
    $data = $presenter->transform($userlist,new UserTransformer());//不包含引用资源,输出集合结构
    print_r($data);
    ```

    **注意transform()方法没有专递第三个参数，第三个参数默认为true，表示处理集合**

  - 输出集合资源，且引用其它资源结构

    和单资源一样，也有两种方式，两种方式代码如下

    ```php
    //方式1
    $data = $presenter->transform($userlist,new UserTransformer(['posts']));//包含引用记录,输出集合结构
    print_r($data);
    
    //方式2
    //包含引用记录,输出集合结构的另一种方式
    $userTransformer = new UserTransformer;
    $userTransformer->setAvailableIncludes('posts');
    $data = $presenter->transform($userlist,$userTransformer);//包含引用记录,输出集合结构
    print_r($data);
    ```

如果你对封装的源码感兴趣，可以[点击这里](https://github.com/fanqingxuan/presenter)查看,或者在github搜索[fanqingxuan/presenter](https://github.com/fanqingxuan/presenter)。