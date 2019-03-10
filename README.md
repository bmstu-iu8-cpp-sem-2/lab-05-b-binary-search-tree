# Лабораторная работа 5

## Задание

1. Реализуйте класс бинарного дерева поиска.

Описание бинарного дерева поиска можно найти в книге Кормена "Алгоритмы. Построение и анализ (3-е издание)", стр 319, глава 12.

```cpp
template <class T>
class BinarySearchTree
{
public:
    struct Node
    {
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

3. Реализуйте функцию нахождения максимальной глубины дерева.
```cpp
template <class T>
unsigned int Depth(const BinarySearchTree<T>&);
```
Если необходимо, сделайте функцию членом класса `BinarySearchTree`.

4. Реализйте фунцию, которая проверяет явяется ли произвольное дерево бинарным деревом поиска.
```cpp
struct TreeNode
{
    Node* Parent;
    Node* Left;
    Node* Right;
    T Value;
};

template <class T>
bool IsBinarySearchTree(Node<T>* root);
```

5. Реализуйте модульные тесты для всех предыдущих заданий. Покрытие кода тестами должно быть не менее 75%.
