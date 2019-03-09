#### 单例模式（Singleton）
在应用程序调用的时候，只能获得一个对象实例。  
```php
<?php

namespace DesignPatterns\Creational\Singleton;

final class Singleton
{
    /**
    * @var Singleton
    */
    private static $instance;

    /**
    * 通过懒加载获得实例（在第一次使用的时候创建）
    */
    public static function getInstance(): Singleton
    {
        if (null === static::$instance) {
            static::$instance = new static();
        }

        return static::$instance;
    }

    /**
    * 不允许从外部调用以防止创建多个实例
    * 要使用单例，必须通过 Singleton::getInstance() 方法获取实例
    */
    private function __construct()
    {
    }

    /**
    * 防止实例被克隆（这会创建实例的副本）
    */
    private function __clone()
    {
    }

    /**
    * 防止反序列化（这将创建它的副本）
    */
    private function __wakeup()
    {
    }
}
```
#### 多例模式（Multiton）
单例模式的推广,有多个一样的实例，每个实例可以有名字  
```php
<?php

namespace DesignPatterns\Creational\Multiton;

final class Multiton
{
    const INSTANCE_1 = '1';
    const INSTANCE_2 = '2';

    /**
     * @var 实例数组
     */
    private static $instances = [];

    /**
     * 这里私有方法阻止用户随意的创建该对象实例
     */
    private function __construct()
    {
    }

    public static function getInstance(string $instanceName): Multiton
    {
        if (!isset(self::$instances[$instanceName])) {
            self::$instances[$instanceName] = new self();
        }

        return self::$instances[$instanceName];
    }

    /**
     * 该私有对象阻止实例被克隆
     */
    private function __clone()
    {
    }

    /**
     * 该私有方法阻止实例被序列化
     */
    private function __wakeup()
    {
    }
}
```
#### 静态工厂模式（Static Factory）
只使用一个静态方法来创建所有类型对象  
```php
<?php

namespace DesignPatterns\Creational\StaticFactory;

/**
 * 注意点1: 记住，静态意味着全局状态，因为它不能被模拟进行测试，所以它是有弊端的
 * 注意点2: 不能被分类或模拟或有多个不同的实例。
 */
final class StaticFactory
{
    /**
    * @param string $type
    *
    * @return FormatterInterface
    */
    public static function factory(string $type): FormatterInterface
    {
        if ($type == 'number') {
            return new FormatNumber();
        }

        if ($type == 'string') {
            return new FormatString();
        }

        throw new \InvalidArgumentException('Unknown format given');
    }
}
```
#### 简单工厂模式（Simple Factory）
可以有多个不同的实例工厂，创建不同的实例  
```php
<?php

namespace DesignPatterns\Creational\SimpleFactory;

class SimpleFactory
{
    public function createBicycle(): Bicycle
    {
        return new Bicycle();
    }
}
```
#### 原型模式（Prototype）
先创建一个原型类，然后可以使用clone创建无数个  
```php

<?php

namespace DesignPatterns\Creational\Prototype;

abstract class BookPrototype
{
    /**
    * @var string
    */
    protected $title;

    /**
    * @var string
    */
    protected $category;

    abstract public function __clone();

    public function getTitle(): string
    {
        return $this->title;
    }

    public function setTitle($title)
    {
        $this->title = $title;
    }
}
class BarBookPrototype extends BookPrototype
{
    /**
    * @var string
    */
    protected $category = 'Bar';

    public function __clone()
    {
    }
}
```
#### 对象池模式（Pool）
一个管理对象池的类
```php
<?php

namespace DesignPatterns\Creational\Pool;

class WorkerPool implements \Countable
{
    /**
    * @var StringReverseWorker[]
    */
    private $occupiedWorkers = [];

    /**
    * @var StringReverseWorker[]
    */
    private $freeWorkers = [];

    //获取
    public function get(): StringReverseWorker
    {
        if (count($this->freeWorkers) == 0) {
            $worker = new StringReverseWorker();
        } else {
            $worker = array_pop($this->freeWorkers);
        }

        $this->occupiedWorkers[spl_object_hash($worker)] = $worker;

        return $worker;
    }

    //归还
    public function dispose(StringReverseWorker $worker)
    {
        $key = spl_object_hash($worker);

        if (isset($this->occupiedWorkers[$key])) {
            unset($this->occupiedWorkers[$key]);
            $this->freeWorkers[$key] = $worker;
        }
    }

    public function count(): int
    {
        return count($this->occupiedWorkers) + count($this->freeWorkers);
    }
}

class StringReverseWorker
{
    /**
    * @var \DateTime
    */
    private $createdAt;

    public function __construct()
    {
        $this->createdAt = new \DateTime();
    }

    public function run(string $text)
    {
        return strrev($text);
    }
}
```
#### 抽象工厂模式（Abstract Factory）
在不指定具体类的情况下创建一系列相关或依赖对象。 通常创建的类都实现相同的接口。   
抽象工厂的客户并不关心这些对象是如何创建的，它只是知道它们是如何一起运行的。  
```php
<?php

namespace DesignPatterns\Creational\AbstractFactory;

/**
 * 在这种情况下，抽象工厂是创建一些组件的契约
 * 在 Web 中。 有两种呈现文本的方式：HTML 和 JSON
 */
abstract class AbstractFactory
{
    abstract public function createText(string $content): Text;
}
class JsonFactory extends AbstractFactory
{
    public function createText(string $content): Text
    {
        return new JsonText($content);
    }
}
abstract class Text
{
    /**
     * @var string
     */
    private $text;

    public function __construct(string $text)
    {
        $this->text = $text;
    }
}
class JsonText extends Text
{
    // 你的逻辑代码
}
```
####  建造者模式（Builder）
将建造的方法统一到一个类中
```php
<?php

namespace DesignPatterns\Creational\Builder;

use DesignPatterns\Creational\Builder\Parts\Vehicle;

/**
 * Director 类是建造者模式的一部分。 它可以实现建造者模式的接口
 * 并在构建器的帮助下构建一个复杂的对象
 *
 * 您也可以注入许多构建器而不是构建更复杂的对象
 */
class Director
{
    public function build(BuilderInterface $builder): Vehicle
    {
        $builder->createVehicle();
        $builder->addDoors();
        $builder->addEngine();
        $builder->addWheel();

        return $builder->getVehicle();
    }
}
$carBuilder = new CarBuilder();
$newVehicle = (new Director())->build($carBuilder);
```
#### 工厂方法模式（Factory Method）
这种模式是「真正」的设计模式， 因为他实现了S.O.L.I.D原则中「D」的 「依赖倒置」。  
```php
$loggerFactory = new StdoutLoggerFactory();
$logger = $loggerFactory->createLogger();
```
### 参考
https://learnku.com/docs/php-design-patterns/2018