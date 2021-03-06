# Полиморфизм

Полиморфизм описывает способ поведения объектов, при котором один и тот же объект ведет себя как различные типы. Описанные раньше приемы делают это возможным: экземпляр класса может быть использован как экземпляр суперкласса, или любого реализованного им интерфейса, без знания его действительного типа.

Логичным представляется реализовать различное поведение экземпляра типа, в зависимости от того как к нему обращаются. Эту концепцию не легко объяснить, поэтому я начну с примера, что может быть если вы изначально не ориентировались на это:

```csharp
class SuperClass : GLib.Object {
    public void method_1() {
        stdout.printf("SuperClass.method_1()\n");
    }
}

class SubClass : SuperClass {
    public void method_1() {
        stdout.printf("SubClass.method_1()\n");
    }
}
```

Эти два класса реализуют методы "method\_1" и "SubClass", то есть содержит два метода называемые "method\_1", который наследуется от "SuperClass". Каждый из них может быть вызван следующим образом:

```csharp
SubClass o1 = new SubClass();
o1.method_1();
SuperClass o2 = o1;
o2.method_1();
```

Это приведет к вызову двух разных методов. Во второй строке о1 считается объектом типа SubClass и вызовется версия метода из этого класса. В четвертой строке о2 имеет тип SuperClass и вызовется метод из него.

Проблема в том, что всякая ссылка на объект типа SuperClass будет вызывать методы описанные именно в базовом классе, даже если объект фактически является экземпляром потомка. Этого можно избежать используя виртуальные методы. Рассмотрим переписанную версию последнего примера:

```csharp
class SuperClass : GLib.Object {
    public virtual void method_1() {
        stdout.printf("SuperClass.method_1()\n");
    }
}

class SubClass : SuperClass {
    public override void method_1() {
        stdout.printf("SubClass.method_1()\n");
    }
}
```

Если данный код использовать как до этого, то `method_1` из `SubClass` будет вызван дважды, потому что мы сказали системе, что `method_1` виртуальный и если он будет переопределен в потомке, то при вызове из потомка будет работать последний вариант.

Данный подход должен быть знаком программистам, пишущим на С++, но он отличается от того, что принято в языках подобным Java, где нужно предпринять некоторые меры, чтобы метод не был `virtual`.

Вы, наверное, так же заметили, что если метод объявлен как `abstract`, то он должен быть и `virtual`. Иначе не получится его вызвать. При реализации такого метода в классе-наследнике вы можете сделать это как переопределение, таким образом оставляя метод виртуальным и наследники и дальше смогут переопределять этот метод под себя.

Так же можно реализовать методы интерфейса таким образом, чтобы классы-потомки могли их переопределять. В данном случае при реализации методов они объявляются как virtual, а затем могут быть переопределены в классах-наследниках.

При создании класса обычно возникает желание использовать код, существующий в базовом классе. Это затруднительно, если одно и то же название метода используется много раз в иерархии классов. При таких случаях в Vala используется ключевое слово **`base`**. Чаще он применяется в случае если вы переопределяете виртуальный метод и одновременно хотите вызвать его в том виде, в котором он присутствует в родительском классе. Следующий пример показывает рассматриваемый случай:

```csharp
public override void method_name() {
    base.method_name();
    extra_task();
}
```

Vala also allows properties to be virtual:

```csharp
class SuperClass : GLib.Object {
    public virtual string prop_1 {
        get {
            return "SuperClass.prop_1";
        }
    }
}

class SubClass : SuperClass {
    public override string prop_1 {
        get {
            return "SubClass.prop_1";
        }
    }
```

  


