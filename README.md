# Семинар 2

Данная работа направлена на изучение структур данных типа Hash Table.
В рамках данной работы я выполнил следующие задания:

- [ ] Освежил память в части хэш-таблиц, поработав с [симулятором заполнения хэш-таблицы](https://www.cs.usfca.edu/~galles/visualization/OpenHash.html)
- [ ] Поэкспериментировал с хэш-таблицами в [симуляторе кастомных хэш-функций](https://iswsa.acm.org/mphf/openDSAPerfectHashAnimation/perfectHashAV.html). Здесь можно выбрать разные хэш-функции, методы разрешения коллизий и т.д.
- [ ] Прочитал [полезную статью](https://habr.com/ru/companies/ruvds/articles/747084/), чтобы дальше было проще создать свою хэш-функцию
- [ ] Сделал отчёт о проделанной работе

Выбрал свой метод разрешения коллизий:
- N % 4 = 2 - **Открытая адресация: квадратичное пробирование** (так как у меня вариант 26 ==> N=26)


Далее я придумал любой класс данных и в там соблюдаются два условия:
1) Я придумали его **сам**!
2) В нём есть хотя бы одно поле типа **int** и одно типа **string** 
Только из названия вашего типа данных, должно быть понятно, что это. То есть типа class Student с номером зачётки и именем, или class Car и т.д.

Далее я реализовал классическую хэш-таблицу, со следующими параметрами:

- [ ] Метод вставки элемента
- [ ] Метод поиска элемента
- [ ] Метод удаления элемента
- [ ] Метод вывода хэш-таблицы
- [ ] Сама хэш-функция (конечно же))))))

Самый главный параметр оценки моей работы - это оригинальность вашей хэш-функции, то есть в итоге получилась функция, которую я написал сам (так как код большой, то для большего понимания я также написал в самом коде много пояснений)
Далее приведу сам код:

```
#include <iostream>
#include <vector>
#include <string>
//Изначально можно было также использовать сетлокейт для русского языка, но... английский круче)

// Класс Book представляет данные о книге
class Book {
public:
    int isbn;           // Уникальный идентификатор книги
    std::string title;  // Название книги
    std::string author; // Автор книги

public:
    // Конструктор (не Lego) с параметрами по умолчанию
    // -1 используется, так как для ISBN это пустая ячейка
    Book(int isbn = -1, const std::string& title = "", const std::string& author = "")
        : isbn(isbn), title(title), author(author) {
    }

    // Проверяет, является ли ячейка пустой (используется для поиска свободных мест)
    bool isEmpty() const { return isbn == -1; }

    // Проверяет, была ли запись удалена (важно для корректного поиска)
    bool isDeleted() const { return isbn == 0; }

    // Геттеры для доступа к полям
    int getISBN() const { return isbn; }
    std::string getTitle() const { return title; }
    std::string getAuthor() const { return author; }
};

class HashTable {
private:
    std::vector<Book> table; // Основной массив для хранения данных
    size_t count;           // Количество элементов в таблице

    // Основная хэш‑функция вычисляет начальный индекс
    // Использует деление по модулю для распределения ключей
    size_t hash(int key) const {
        return static_cast<size_t>(key) % table.size();
    }

public:
    // Конструктор создаёт таблицу заданного размера
    // Все ячейки инициализируются пустыми книгами (isbn = -1)
    HashTable(size_t capacity = 16) : table(capacity, Book()), count(0) {}

    // Вставка элемента в хэш‑таблицу
    bool insert(const Book& book) {
        // Если таблица заполнена более чем на 50 %, выполняем рехэширование
        if (count >= table.size() / 2) {
            resize();
        }

        size_t initialIndex = hash(book.getISBN()); // Начальный индекс по хэш‑функции
        size_t index = initialIndex;             // Текущий индекс для проверки
        size_t k = 0;                           // Номер попытки (k = 0, 1, 2, ...)

        // Константы для квадратичного пробирования
        // c1 и c2 влияют на шаг смещения при коллизии
        const size_t c1 = 1;
        const size_t c2 = 3;

        // Цикл поиска свободной ячейки или совпадения ISBN
        while (!table[index].isEmpty() && !table[index].isDeleted()) {
            // Если программа нашла книгу с таким же ISBN — обновляет данные
            if (table[index].getISBN() == book.getISBN()) {
                table[index] = book;
                return true;
            }

            // Квадратичное пробирование:
            // index = (initialIndex + c1*k + c2*k²) mod size
            ++k;
            index = (initialIndex + c1 * k + c2 * k * k) % table.size();

            // Защита от бесконечного цикла: если прошли все ячейки,
            // значит таблица почти заполнена — выполняем рехэширование
            if (k >= table.size()) {
                resize();
                return insert(book); // Повторяем вставку в увеличенной таблице
            }
        }

        // Программа нашла свободную ячейку — вставляет книгу
        table[index] = book;
        ++count; // Программа увеличивает счётчик элементов
        return true;
    }

    // Поиск элемента по ISBN
    bool find(int isbn, Book& result) const {
        size_t initialIndex = hash(isbn); // Начальный индекс
        size_t index = initialIndex;
        size_t k = 0;
        const size_t c1 = 1;
        const size_t c2 = 3;

        // Ищет книгу, пока не встретит пустую ячейку
        while (!table[index].isEmpty()) {
            // Если ячейка не удалена и ISBN совпадает — Well done
            if (!table[index].isDeleted() && table[index].getISBN() == isbn) {
                result = table[index]; // Копирует данные в результат
                return true;
            }

            // Сдвигается по квадратичной формуле
            ++k;
            index = (initialIndex + c1 * k + c2 * k * k) % table.size();

            //Если все позиции проверили, но ничего не нашли
            if (k >= table.size()) break;
        }
        return false; // Книга не найдена
    }

    // Удаление элемента по ISBN
    bool remove(int isbn) {
        size_t initialIndex = hash(isbn);
        size_t index = initialIndex;
        size_t k = 0;
        const size_t c1 = 1;
        const size_t c2 = 3;

        while (!table[index].isEmpty()) {
            // Если нашла нужную книгу и она не помечена как удалённая
            if (!table[index].isDeleted() && table[index].getISBN() == isbn) {
                // Помечаем как удалённую: isbn = 0
                // Это важно для корректной работы поиска 
                table[index] = Book(0, "", "");
                --count; // Уменьшаем счётчик элементов
                return true;
            }

            // Продолжаем поиск по квадратичной формуле
            ++k;
            index = (initialIndex + c1 * k + c2 * k * k) % table.size();

            // Защита от бесконечного цикла
            if (k >= table.size()) break;
        }
        return false; // Книга не найдена для удаления
    }

    // Вывод содержимого таблицы
    void print() const {
        std::cout << "\n--- Hash Table (Quadratic Probing) ---\n";
        for (size_t i = 0; i < table.size(); ++i) {
            const Book& book = table[i];
            if (!book.isEmpty()) { // Если ячейка не пустая
                if (book.isDeleted()) {
                    // Показывает, что ячейка удалена
                    std::cout << "[" << i << "] DELETED\n";
                }
                else {
                    // Программа выводит данные книги
                    std::cout << "[" << i << "] ISBN: " << book.getISBN()
                        << ", Title: '" << book.getTitle()
                        << "', Author: '" << book.getAuthor() << "'\n";
                }
            }
        }
        std::cout << "--- End of Table ---\n\n";
    }

private:
    // Рехэширование, т.е. увеличение размера таблицы и перераспределение элементов
    void resize() {
        // Сохраняем старую таблицу во временный вектор
        std::vector<Book> oldTable = std::move(table);

        // Очищает текущую таблицу и увеличивает её размер в 2 раза
        table.clear();
        table.resize(oldTable.size() * 2, Book());

        // Обнуляем счётчик элементов
        count = 0;

        // Вставляет все непустые и не удалённые записи из старой таблицы
        for (const Book& book : oldTable) {
            if (!book.isEmpty() && !book.isDeleted()) {
                insert(book);
            }
        }
    }
};

int main() {
    HashTable library(4); // Маленькая таблица для демонстрации коллизий (несколько тестов)

    // Тест 1 (вставка с коллизиями)
    library.insert(Book(1, "Book A", "Author X"));
    library.insert(Book(9, "Book B", "Author Y")); // 9 % 4 = 1 это коллизия с ISBN 1
    library.insert(Book(5, "Book C", "Author Z")); // 5 % 4 = 1 это ещё коллизия

    library.print();

    // Тест 2 (поиск)
    Book found;
    if (library.find(5, found)) {
        std::cout << "Found: " << found.getTitle() << "\n";
    }

    // Тест 3 (удаление и повторная вставка)
    library.remove(9);
    library.insert(Book(13, "Book D", "Author W")); // Должен занять ячейку от удалённой книги
    library.print();

    return 0;
}

```
Скриншот из [симулятора кастомных хэш-функций](https://iswsa.acm.org/mphf/openDSAPerfectHashAnimation/perfectHashAV.html).
<img width="1036" height="875" alt="image" src="https://github.com/user-attachments/assets/025cad4e-62cd-4035-abcd-c44278127952" />
