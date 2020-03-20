# Лабораторная работа 5

## Цель

Данная работа позволит студенту:
* получить навыки реализации структур данных
* углубить свои знания об исключениях в С++
* получить навыки обработки исключений
* улучшить понимание зачем и когда необходимо использовать механих исключений

## Теоретическая часть

### Спецификации исключений

Часто спецификацию исключений не используют, а используют же только один ограниченный случай. С упором на этот случай мы и рассмотрим данную тему.

Спецификации исключений — это механизм объявления функций с указанием того, будет ли функция генерировать исключения (и какие именно) или нет. Это может быть полезно при определении необходимости помещения вызова функции в блоке try.

Например, можно объявить функцию:
```cpp
float div(float a, float b) throw(std::runtime_error);
```
Это сообщает, что функция `div` может генерировать исключение типа `std::runtime_error`.

Или например,
```cpp
float bar(float a, float b) throw(...);
```
что сообщает, что могут генерироваться любые исключения.

Два данных случая используются крайне редко при разработке на C++.

Следующий случай используется часто и имеет практическую ценность.


### Спецификатор `throw()`

Предположим есть функция, которая не должна выбрасывать исключений. А если функция и генерирует исключения, то обработка исключений происходит внутри фукнции.

Рассмотрим простой геттер
```cpp
size_t size() {
  return sz;
}
```

Данная функция не может генерировать исключения. О таком её свойстве необходимо сообщить компилятору. Для этого необходиом дописать спецификатор `throw()`.
```cpp
size_t size() throw() {
  return sz;
}
```

Т.к. компилятор теперь знает, что эта фукнция **не должна** генерировать никакие исключения, то компилятор может использовать это для оптимизаций производительности программы.

Кроме таких простых случаев, спецификатором `throw()` могут обладать более сложные фукнции, но которые не выбрасывают исключений наружу. Например,
```cpp
size_t foo() throw() {
  try {
    float a = 0;
    float b = 0;
    std::cin >> a >> b;
    std::cout << div(a, b);
  } catch (...) {
    std::cout << "catch ...";
  }
}
```
Функция выше самостоятельно перехватывает все сгенерированные исключения. Значит она не выбрасывает ни одно исключение наружу. И её награждаем спецификатором `throw()`, чтобы компилятор мог лучше оптимизировать программу.

Таким образом, `throw()` гарантирует, что исключения не могут покидать функцию. Но что произойдет, если какое-то исключение покинет функцию? Если функция, объявленная с использованием `throw()`, попытается сгенерировать исключение, то, согласно стандарту, будет вызвана фунция [`unexpected()`](https://en.cppreference.com/w/cpp/error/unexpected). Функция `unexpected()` по умолчанию аварийно завершает программу.

Спецификатор `throw()` используется для функций, которые принципиально не генерируют никакие исключения, которые выходят за пределы функции. Это помогает компилятору лучше оптимизировать исходный код программы. Если исключение покидает функцию со спецификатором `throw()`, то программа аварийно завершится.

Поэтому `throw()` надо использовать для функций, которые гарантированно не бросают исключения наружу. Либо в тех случаях, когда выбрашенное исключение из фукнции равносильно завершению программы, т.е. тогда, когда исправлять возникшую ситуацию уже бесмыслено, и лучшее решение - это завершение всей программы (чтобы не стало хуже).

### Спецификатор `noexcept`
Со стандартом С++11 появился спецификатор времени компиляции `noexcept`, которой говорит компилятору о том, что функция не будет выбрасывать исключения. Этот оператор очень похож на спецификатор `throw()` и выполняет такую же роль.

**Рекомендуется** использовать новый спецификатор `noexcept` вместо `throw()`!

Спецификатор времени компиляции `noexcept` позволяет компилятору генерировать еще более производительнй код и сильно уменьшает размер итогового файла.

__Заметка__
Более производительный код получается за счет применения move assignment и move construction. Согласно стандарту, стандартные алгоритмы и контейнеры НЕ должны использовать move assignment и move construction, если конструктор перемещения и оператор перемещения могут кидать исключения.


Рассмотрим отличия `noexcept` и `throw()`.

#### Первое отличие

Если функция помеченная `noexcept` выпустит исключение наружу, то программа вызовет `std::terminate()` и завершится.

Если функция помеченная `throw()` выпустит исключение наружу, то программа вызовет `std::unexpected()`.

Главное отличие в этих случаях, в том, что вызов функции `std::terminate()` не вызвает деструкторы для уже созданных переменных.

#### Второе отличие
Спецификатор `noexcept` удобно применять при разработке шаблонов классов и фукций. Например, вы пишите шаблон функцию, которую потенциально можно пометить как `noexcept`. Но вы пишите шаблон функции, поэтому будет ли метод `noexcept` или нет, зависит от свойств конкретного типа, который будет использован в шаблоне.

Спецификатор `noexcept` может принимать значение, которое определяет будет ли применяться `noexcept` к функции или нет. `noexcept(true)` равносилен обычному спецификатору `noexcept`. `noexcept(false)` означает, что `noexcept` не применяется к функции. Конечно же, значение в скобках должно вычисляться во время компиляции программы.

##### Пример
Рассмотрим два класса: класс натуральных чисел с нулем включительно `NaturalNumber`, и натуральных чисел без нуля `NaturalNumberPlus`.
Для первого класса, фукнция `div` может генерировать исключение. Но в случае `NaturalNumberPlus` делитель никогда не может быть нулем, поэтому фукнция `div` не должна генерировать исключение.

Такое поведение можно представить следующим кодом:
```cpp
struct NaturalNumberPlus {
  unsigned int number = 1;
  constexpr static bool maybe_null = false;
  // ...
};

struct NaturalNumber {
  unsigned int number = 0;
  constexpr static bool maybe_null = true;
  // ...
};

template <typename T>
T div(T a, T b) noexcept(!T::maybe_null) {
  if (b.number == 0) {
    throw std::runtime_error("sdf");
  }
  return T {a.i / b.i};
}

NaturalNumberPlus a{1}, b {2};
auto c = div(a, b); // noexcept функция
NaturalNumber x{1}, y{2};
auto z = div(x, y); // НЕ noexcept функция
```

Рассмотрим функцию
```cpp
template <typename T>
T div(T a, T b) noexcept(!T::maybe_null)
```

В спецификатор `noexcept` передается значение `maybe_null` шаблонного класса `T`.

Для класса `NaturalNumberPlus` значение `maybe_null` равно `false`, означаютщее, что число никогда не может быть нулем.

Для класса `NaturalNumber` значение `maybe_null` равно `true`, означаютщее, что число может равняться нулю.

Поэтому функция `div<NaturalNumberPlus>` будет объявлена как `noexcept` функция. Но `div<NaturalNumber>` не является `noexcept` функцией. Это означает, что функция может выбрасывать исключение наружу.

## Вывод

* Используйте `noexcept` вместо `throw()`
* Не забывайте спецификатор `noexcept` у конструктора копирования/перемещения и у операторов присваивания/перемещения. Это поможет повысить производительность программы.
```cpp
MyClass(constMyClass&) noexcept = default;
MyClass(MyClass&&) noexcept = default;
MyClass& operator=(const MyClass&) noexcept = default;
MyClass& operator=(MyClass&&) noexcept = default;
```

* Спецификаторы `noexcept` и `throw()` используются для функций, которые принципиально не генерируют никакие исключения, которые выходят за пределы функции. Это помогает компилятору лучше оптимизировать исходный код программы.
* Если исключение покидает `noexcept` функцию, то вызывается `std::terminate` и программа завершается
* Если исключение покидает `throw()` функцию, то вызывается `std::unexpected` и программа завершается


## Задание

1. Реализуйте класс бинарного дерева поиска.

Описание бинарного дерева поиска можно найти в книге Кормена "Алгоритмы. Построение и анализ (3-е издание)", стр 319, глава 12.

```cpp
template <class T>
class BinarySearchTree {
public:
  struct Node {
    Node* Left;
    Node* Right;
    T Value;
  };

  BinarySearchTree();
  ~BinarySearchTree();

  void Add(const T&);
  Node* Find(const T&);
  void Remove(Node*);
};
```

2. Реализуйте функцию печати бинарного дерева по слоям.
```cpp
template <class T>
std::ostream& operator<< (std::ostream&, const BinarySearchTree<T>&);
```
Если необходимо, сделайте функцию дружественной к классу.

Чтобы реализовать модульные тесты для этого задания воспользуйтесь классом `std::stringstream`.

3. Реализуйте функцию нахождения максимальной глубины дерева.
```cpp
template <class T>
unsigned int Depth(const BinarySearchTree<T>&);
```
Если необходимо, сделайте функцию членом класса `BinarySearchTree`.

4. Реализйте фунцию, которая проверяет явяется ли произвольное дерево бинарным деревом поиска.
```cpp
struct TreeNode {
    Node* Parent;
    Node* Left;
    Node* Right;
    T Value;
};

template <class T>
bool IsBinarySearchTree(Node<T>* root);
```

5. Реализуйте модульные тесты для всех предыдущих заданий. Покрытие кода тестами должно быть не менее 95%.
