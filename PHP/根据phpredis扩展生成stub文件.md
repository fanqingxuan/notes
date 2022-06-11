### PHP
### stub文件
phpredis是php的c扩展形式实现的redis客户端代码，有时候我们想知道这个扩展到底有哪些方法，或者我们想让编辑器帮我们自动提示，这时候就需要stub文件了，当然phpstorm这个强大的编辑器插件里面已经包含了phpredis的stub文件，但是其它如sublime，vscode却没有，这时候我们就需要自己去生成，然后copy到项目里面，就能帮我们提示了

### 启发
研究鸟哥的yaf框架的时候，鸟哥自己写了个yaf-tools，包含了yaf的所有方法，便于编辑器提示，鸟哥在工具里面还给出了生成yaf stub文件的代码，所以拿鸟哥的代码改成phpredis扩展的，就可以生成phpredis stub了

### 实现
gen-redis-stub.php文件代码
```php
<?php
/**
 * Redis Classes stub generator
 *
 * @author  Json
 * @date    2020-05-07 16:49:09
 * @version $Id$
 */

$classPrefix = "Redis";
$classes = array_merge(get_declared_classes(), get_declared_interfaces());
foreach ($classes as $key => $value) {
    if (strncasecmp($value, $classPrefix, 5)) {
        unset($classes[$key]);
    }
}
ob_start();
echo "<?php\n";

foreach ($classes as $class_name) {
    $class = new ReflectionClass($class_name);
    $indent  = "";

    $classAttributes = "";

    if ($class->isInterface()) {
        $classAttributes .= "interface ";
    } else {
        if ($class->isFinal()) {
            $classAttributes .= "final ";
        }

        if ($class->isAbstract()) {
            $classAttributes .= "abstract ";
        }

        $classAttributes .= "class ";
    }

    echo $indent, $classAttributes, $class_name;

    /* parent */
    $parent = $class->getParentClass();
    if ($parent) {
        echo " extends ", $parent->getName();
    }

    /* interface */
    $interfaces = $class->getInterfaceNames();
    if (count($interfaces)) {
        echo " implements ", join(", ", $interfaces);
    }
    echo " {\n";

    $indent .= "\t";
    /* constants */
    $constants = $class->getConstants();
    if (0 < count($constants)) {
      echo $indent, "/* constants */\n";

      foreach ($constants as $k => $v) {
           echo $indent, "const ", $k , " = \"", $v , "\";\n";
      }
      echo "\n";
    }

    /* properties */
    $properties = $class->getProperties();
    if (0 < count($properties)) {
      echo $indent, "/* properties */\n";
      $values     = $class->getDefaultProperties();
      foreach ($properties as $p) {
          echo $indent;

          if ($p->isStatic()) {
              echo "static ";
          }

          if ($p->isPublic()) {
              echo "public ";
          } else if ($p->isProtected()) {
              echo "protected ";
          } else {
              echo "private ";
          }

          echo '$', $p->getName(), " = ";

          if (isset($values[$p->getName()])) {
              echo '"', $values[$p->getName()], '"';
          } else {
              echo "NULL";
          }
          echo ";\n";
      }
      echo "\n";
    }

    /* methods */
    $methods = $class->getMethods();
    if (0 < count($methods)) {
      echo $indent, "/* methods */\n";

      foreach ($methods as $m) {
          echo $indent;
          echo implode(' ', Reflection::getModifierNames($m->getModifiers()));
          echo " function ", $m->getName(), "(";

          if ($m->isAbstract()) {
              // abstract methods are without a body "{ ... }"
              echo ");\n";
              continue;
          }

          $parameters = $m->getParameters();
          $number = count($parameters);
          $index  = 0;
          foreach ($parameters as $a) {
              if (($type = $a->getClass())) {
                  echo $type->getName(), " ";
              } else if ($a->isArray()) {
                  echo "array ";
              }

              if ($a->isPassedByReference()) {
                  echo "&";
              }

              $name = $a->getName();
              if ($name == "...") {
                  echo '$_ = "..."';
              } else {
                  echo "$", $name;
              }

              if ($a->isOptional()) {
                  if ($a->isDefaultValueAvailable()) {
                      echo " = ", $a->getDefaultValue();
                  } else {
                      echo " = NULL";
                  }
              }

              if (++$index < $number) {
                  echo ", ";
              }
          }

          echo ") {\n";
          echo $indent, "}\n";
      }
    }

    $indent = substr($indent, 0, -1);
    echo $indent, "}\n";

}


$content = ob_get_contents();
ob_end_clean();

file_put_contents("Redis.stub.php",$content);

```
### 使用
```php
$ php gen-redis-stub.php
```
会在当前目录生成Redis.stub.php

**其实这个例子的目的不是教大家生成redis的stub文件，而是教大家如何灵活使用php中的反射相关的各种方法**