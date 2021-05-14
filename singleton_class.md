# What is Singleton Class in Ruby

It has many names: metaclass, engenclass, singleton class 

---
## Why we need Singleton Class 
It solve the `method dispatching` problem 

Assuming we have the following `Vehicle Class` 
```Ruby
class Vehicle 

  def initialize(speed)
    @speed = speed
  end 

  def drive
    puts 'drive'
  end
end

car = Vehicle.new(25)
car.drive 
```

In memory, ruby present it like 

![diagram_1](images/singleton_1.png)

To find out which method implementation is called for any object, let's assuming complier will follow the klass pointer. 

When you call `car.drive`, you compiler will just find the klass of your `car` instance, which is `Verchile`, and find `drive` method there. It works

But if we make the `Vehicle` a bit more complicated 

```Ruby
class Vehicle 
  @@factory = []

  def self.register(vehicle) 
    factory << vehicle 
  end
  
  def initialize(speed)
    @speed = speed
  end 

  def drive
    puts 'drive'
  end
end

car = Vehicle.new(25)
Vehicle.register(car) 
```

How would ruby represent that in memory ?  
![diagram_2](images/singleton_2.png)

--- 
## Discover Singleton class 
Singleton class solved this problem. From now, I will represent the singleton class of class `A` as `#A`, singleton class of instance `b` as `#b`.

![diagram_3](images/singleton_3.png)

Now when you do `Vehicle.register(car) `, if will find the klass of `Vehicle`, which is `#Vehicle`, and find the `register` method there. 
When you do `Vehicle.new`, it will follow the `klass` pointer, until we find the `new` method in `Class` class
Everything works ! (sorry this is a lie, we will see it a bit later) 

Something interesting fact here: 
* Singleton class is created only when needed
* You can use `singleton_class` method to get the singleton_class of a class  
* The `class` of `singleton_class` is also `Class` 
* When you do `def self.register`, ruby acutally open the singleton class of `Vehicle`, and add a register method there. 

---
## Singleton class for instance 

The same logic metioned apply for instances
```
car = Vehicle.new(25)
def car.start
  puts 'start'
end
```

In memory, ruby represent it like this: 

![diagram_4](images/singleton_4.png)

---
## The turth about Singleton Class 

Now let's look at a more complicated Vehicle example 
```Ruby
class Vehicle 
  @@factory = []
  
  def self.register(vehicle)
    @@factory << vehicle
  end
  
  def initialize(speed)
    @speed = speed
  end 

  def drive
    puts 'drive'
  end
end

class Car < Vehicle 
  def self.register(car)
    super
  end  
end 

car = Car.new(25)
Car.register(car)  
```

How would ruby reprensent that in memory ? 
![diagram_5](images/singleton_6.png)

Now just follow the `klass` pointer, `Car` can never find the `register` method. There must be a bridge between `#Car` and `#Vehicle` 

Here is the bridge ! 
![diagram_6](images/singleton_6.png)

The truth is, in order to find the method implentation of a calling object, Ruby Complier will first find the `klass` of the object, check the method exist. If not, it will follow the `super` chain up and repeat the same process, until find the method. 

Quick Proof 
```
class A
  def self.say_hi; end
end 

class B < A
  def self.say_hi
    super
  end
end

B.super # => A
B.singleton_class.superclass # => #<Class:A>
A.singleton_class.super_class # => #<Class:Object> 

---
## Alternative syntaxes 

We can use `class << ` syntax to open singleton class and add method to it 

```Ruby 
car = Vehicle.new(2000)
class << car  
  def start
    "starting..."
  end
end
```

We are using this pattern when define class methods in a class 
```
class A
  class << self  # self is A 
    def class_method_1 ; 
    end
  end
end
```

Another way is using `extend`. When you do `self.extend(some_module)`, what it actually doing is open the singletone class of `self`, and append methods from module to it. 

  