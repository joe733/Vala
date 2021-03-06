# Слабые ссылки\(Weak References\)

Управление памятью в Vala основано на подсчете ссылок. Каждый раз когда объект присваивается переменной его внутренний счетчик увеличивается на 1, и каждый раз, когда такие переменные выходят из области видимости внутренний счетчик уменьшается на 1. Если счетчик дойдет до 0, объект будет удален.

Однако существует вероятность возникновения циклических ссылок в структурах данных. Например в дереве узлы хранят ссылку на родителя, или в двойных связанных списках каждый элемент хранит ссылку на предыдущий элемент, а он - на следующий.

В подобных случаях объекты обязаны хранить ссылки друг на друга, даже если они должны быть удалены. Для устранения подобных циклических ссылок используйте модификатор weak для одной из ссылок:

```csharp
class Node {
    public weak Node prev;
    public Node next;
}
```

Эта тема раскрыта более детально на странице: Vala's Memory Management Explained.

## Владение\(Ownership\)

### Бесхозяйные ссылки

Обычно при создании объекта в Vala вам возвращается ссылка не него. Точнее помимо передачи ссылки, объект еще у себя создает запись о существовании указателя. Так же при создании другого указателя происходит аналогичное создание записи. Так как объект знает, сколько на него ссылается, он может быть удален при возникновении необходимости. Это основа [управления памятью в Vala](https://wiki.gnome.org/Projects/Vala/ReferenceHandling).

### **Methods ownership**

Бесхозяйные ссылки, наоборот, не создают записи в объекте, на который ссылаются. Это позволяет удалять объект, когда это кажется целесообразным, несмотря на то, что могут существовать ссылки на него. Обычно это достигается через определение метода, возвращающего бесхозяйную ссылку, напр.:

```csharp
class Test {
    private Object o;

    public unowned Object get_unowned_ref() {
        this.o = new Object();
        return this.o;
    }
}
```

Когда вызывается данный метод вы ожидаете, что он вернет слабую ссылку:

```csharp
unowned Object o = get_unowned_ref();
```

Однако это все кажется чересчур усложненным из-за концепции владения.

* Если объект о не хранился бы в классе, то при возврате методом `get_unowned_ref` он бы стал бесхозяйным \(т.е. не было бы ссылок на него\). В таком случае объект был бы удален и метод никогда бы не возвращал допустимую ссылки.
* Если бы возвращаемое значение не было обозначено как unowned владение перешло бы к вызывающему коду. А он ожидает бесхозяйную ссылку, которой не возможно присвоить владения.

Если вызывающий код написан как

```csharp
Object o = get_unowned_ref();
```

Vala попытается получить ссылку или копию экземпляра, на который ссылается бесхозяйная ссылка.

### **Properties ownership**

В отличие от нормальных методов свойства всегда возвращают бесхозяйные значения. Это значит вы не можете вернуть созданный объект через метод get. Это так же значит, что вы не можете использовать возвращенное из метода не бесхозяйное значение. Дело в том, что значение свойства принадлежит объекту, у которого это свойство объявлено. Вызов этого свойства не должен забирать или копировать \(через дублирование или инкрементом счетчика\) его значение из под владения объекта, к которому это свойство принадлежит.

Таким образом, следующий пример вызовет ошибку компиляции

```csharp
public Object property {
    get {
        return new Object();   // НЕПРАВИЛЬНО: свойство возвращает бесхозяйную ссылку,
                               // созданный объект будет удален, когда
                               // область видимости геттера заканчивается, вызывающий этот
                               // геттер код перестает получать неправильные ссылки
                               // на удаленный объект
    }
}
```

и нельзя сделать вот так:

```csharp
public string property {
    get {
        return getter_method();   // НЕПРАВИЛЬНО: по той же причине, что выше.
    }
}

public string getter_method() {
    return "some text"; // метод дублирует и возвращает "какой-то текст" в этой точке.
}
```

с другой стороны, все прекрасно работает

```csharp
public string property {
    get {
        return getter_method();   // GOOD: getter_method returns an unowned value
    }
}

public unowned string getter_method() {
    return "some text";
    // Не беспокойтесь, что текст не присваивается никакой сильной переменной.
    // Строковые литералы в Vala всегда принадлежат самому программному модулю,
    // и существуют, пока модуль находится в памяти
}
```

Модификатор unowned может сделать автоматические свойства бесхозяйными. Это значит, что

```csharp
public unowned Object property { get; private set; }
```

идентично с

```csharp
private unowned Object _property;

public Object property {
    get { return _property; }
}
```

Ключевое слово **`owned`** может быть использовано для указания свойству на возврат ссылки с владельцем, так что значение свойства будет представлено на стороне объекта. Дважды подумайте, прежде чем его добавлять. Будет это свойство или просто метод типа `get_xxx`? Так же могут возникнуть проблемы в дизайне вашего кода. Тем не менее, данный код будет правильным,

```csharp
public owned Object property { owned get { return new Object(); } }
```

Бесхозяйные ссылки действуют подобно указателям, которые описаны выше, но использовать их намного проще, т.к. их легко можно превратить в нормальные ссылки. Тем не менее, вы не должны их часто использовать, только если вы уверены, что вы делаете.

### Передача владения **Ownership Transfer**

Ключевое слово owned используется для передачи владения.

Как префикс типа параметра, он указывает на то, что данная ссылка на объект переходит в ведение текущего кода.

Как оператор он позволяет избежать дублирования объектов, не имеющих подсчета ссылок, что обычно невозможно в Vala. Например

```csharp
Foo foo = (owned) bar;
```

Здесь значение _bar_ станет `null` и _foo_ получит ссылку/владение объектом, на который ссылался _bar_.

