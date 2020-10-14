# Spring IOC

IOC：（Inversion of Control）控制反转。 Spring最核心的部分，Spring家族中任何组件的基本。IOC本身并不是一个技术，而是一种思想，他使你从繁琐的对象交互中解脱出来，进而专注于对象本身，更进一步突出面向对象。

要了解IOC首先先了解依赖注入（ Dependency  Inversion）。含义是把底层类作为参数传递给上层类，实现上层对下层的“控制”。

使用依赖注入之前：

```java
public class Tire {
    private int size;
    Tire (int size) {
        this.size = size;
    }
}
public class Bottom {
    private Tire tire;
    Bottom (int size) {
        this.tire = new Tire(size);
    }
}
public class Framework {
    private Bottom bottom;
    Framework (int size) {
        this.bottom = new Bottom(size);
    }
}
public class Luggage {
    private Framework framework;
    Luggage (int size) {
        this.framework = new Framework(size);
    }
    public void move(){}
}
int size = 30;
Luggage luggage = new Luggage(size);
luggage.move();
```

如果需要修改Tire类的构造方法，增加新的属性，则不得不对Bottom、Framework、Luaggage所有依赖Tire的类都进行修改，以适应Tire的变化。这样极大地降低了代码的可维护性，增大了代码之间的耦合性。特别是对大型项目而言，依赖关系错综复杂。

使用依赖注入后：

```java
//依赖注入
public class Tire {
    private int size;
    Tire(int size) {
        this.size = size;
    }
}
public class Bottom {
    private Tire tire;
    Bottom(Tire tire) {
        this.tire = tire;
    }
}
public class Framework {
    private Bottom bottom;
    Framework(Bottom bottom) {
        this.bottom = bottom;
    }
}
public class Luggage {
    private Framework framework;
    Luggage(Framework framework) {
        this.framework = framework;
    }
    public void move(){}
}
int size = 30;
Tire tire = new Tire(size);
Bottom bottom = new Bottom(tire);
Framework framework = new Framework(bottom);
Luggage luggage = new Luggage(framework);
luggage.move();
```

改用依赖注入后，对Tire进行修改并不会影响到其他的类，这样一来代码的可维护性得到了提高，代码之间的耦合度也降低了。

## 依赖注入的方式：

- Setter（设值注入）
- Interface（接口注入）
- Constructor（构造注入）
- Annotation（注解注入）