# C++ Assignment 3 – CO3

**Name:** Lakshmi Narayanan NTN  
**Register Number:** 192524353  
**Subject:** C++ Programming  
**Assignment:** Assignment 3 – CO3

## CO3 Coverage

Construct C++ programs using constructors, destructors and operator overloading to control object creation, destruction and operator behaviour.

Topics covered:
- Default Constructor
- Parameterized Constructor
- Overloaded Constructor
- Copy Constructor
- Dynamic Constructor
- Destructor
- Operator Overloading
- Rules for Operator Overloading

**Nature of Assignment:** Individual

---

## Q1 – Book – Constructors

### Program

```
#include <iostream>
#include <string>
#include <stdexcept>
using namespace std;

class Book {
    string title;
    string author;
    int copies;

    void validate(int c) {
        if (c < 0)
            throw invalid_argument("copies cannot be negative");
    }

public:
    Book() : Book("", "Unknown", 0) {
        cout << "Default ctor\n";
    }

    Book(string t) : Book(t, "Unknown", 0) {}

    Book(string t, string a, int c)
        : title(t), author(a), copies(c) {
        validate(c);
    }

    void show() const {
        cout << title << " | " << author << " | " << copies << "\n";
    }
};

int main() {
    Book b1;
    Book b2("Wings of Fire");
    Book b3("C++", "Bjarne", 5);

    b1.show();
    b2.show();
    b3.show();

    try {
        Book bad("X", "Y", -5);
    }
    catch (exception& e) {
        cout << "Error: " << e.what() << "\n";
    }
}
```

### Theory

- The compiler provides a default constructor only when no constructor is declared.
- If any constructor is declared, the compiler does not automatically provide a default constructor.
- Initializer lists initialize members directly and avoid unnecessary double work.
- Initializer lists are required for initializing const and reference members.
- Initialization order follows the order of member declaration, not the order in the initializer list.
- `-Wreorder` can warn about initialization order issues.
- The `explicit` keyword prevents unwanted implicit conversions in single-argument constructors.
- An alternative is using one constructor with default arguments:

```
Book(string t = "", string a = "Unknown", int c = 0)
```

**Conclusion:** Constructor overloading and delegation are clearer for understanding and viva explanations. Default arguments provide a shorter alternative.

---

## Q2 – Prescription – Copy Constructor and Destructor

### Program

```
#include <iostream>
#include <string>
using namespace std;

class Prescription {
    int n;
    string* medicines;

public:
    explicit Prescription(int count)
        : n(count), medicines(new string[count]) {}

    Prescription(const Prescription& o)
        : n(o.n), medicines(new string[o.n]) {

        for (int i = 0; i < n; i++)
            medicines[i] = o.medicines[i];

        cout << "Copy ctor called\n";
    }

    Prescription& operator=(const Prescription& o) {
        if (this == &o)
            return *this;

        string* tmp = new string[o.n];

        for (int i = 0; i < o.n; i++)
            tmp[i] = o.medicines[i];

        delete[] medicines;

        medicines = tmp;
        n = o.n;

        return *this;
    }

    ~Prescription() {
        delete[] medicines;
    }

    void set(int i, const string& m) {
        medicines[i] = m;
    }

    string get(int i) const {
        return medicines[i];
    }
};

int main() {
    Prescription p(2);
    p.set(0, "Paracetamol");
    p.set(1, "ORS");

    Prescription q = p;
    q.set(1, "Cetirizine");

    Prescription r(1);
    r = p;
    p.set(0, "Ibuprofen");

    cout << p.get(0) << ' ' << p.get(1) << " | "
         << q.get(0) << ' ' << q.get(1) << " | "
         << r.get(0) << ' ' << r.get(1) << "\n";
}
```

### Expected Output

```
Copy ctor called
Ibuprofen ORS | Paracetamol Cetirizine | Paracetamol ORS
```

### Theory

**Why Shallow Copy Crashes**

- Both objects point to the same dynamically allocated array.
- When destructors run, the same memory may be deleted twice.
- This causes a double-free error.

**Shallow Copy:**
Two pointers point to one memory block.

**Deep Copy:**
Two pointers point to two separate memory blocks.

### Rule of Three

If a class needs one of the following, it generally needs all three:

1. Destructor
2. Copy Constructor
3. Copy Assignment Operator

All three manage the same resource.

### When Copy Constructor Is Called

1. `Prescription q = p;`
2. Passing an object by value: `f(p)`
3. Returning an object by value: `return p;`

### Self-Assignment

When `p = p` is performed without a self-assignment guard, the object may delete its own array before copying from it.

The condition below prevents this:

```
if (this == &o)
    return *this;
```

### Alternative – Vector

```
vector<string> medicines;
```

- `vector<string>` automatically manages memory.
- It follows the Rule of Zero.
- The compiler-generated copy operations perform deep copying.
- Time complexity: O(n).
- Space complexity: O(n).
- Manual deep copy has the same complexity.
- Shallow copy is O(1), but unsafe for independently owned dynamic memory.

**Conclusion:** Using `vector` is generally safer in real-world C++ programs.

---

## Q3 – Auditorium – Dynamic Memory Allocation

### Program

```
#include <iostream>
#include <vector>
using namespace std;

class Auditorium {
    int rows, cols;
    char** seats;

public:
    Auditorium(int r, int c) : rows(r), cols(c) {
        if (r <= 0 || c <= 0) {
            seats = nullptr;
            rows = cols = 0;
            cout << "Zero size event\n";
            return;
        }

        seats = new char*[rows];

        for (int i = 0; i < rows; i++) {
            seats[i] = new char[cols];

            for (int j = 0; j < cols; j++)
                seats[i][j] = 'E';
        }

        cout << "Ctor: " << rows << "x" << cols << " allocated\n";
    }

    ~Auditorium() {
        if (seats) {
            for (int i = 0; i < rows; i++)
                delete[] seats[i];

            delete[] seats;
        }

        cout << "Dtor: freed + Summary rows=" << rows << "\n";
    }

    void book(int r, int c) {
        if (r < rows && c < cols)
            seats[r][c] = 'B';
    }

    void cancel(int r, int c) {
        if (r < rows && c < cols)
            seats[r][c] = 'E';
    }
};

int main() {
    Auditorium a(2, 3);

    {
        Auditorium b(2, 2);
    }

    Auditorium* p = new Auditorium(1, 2);
    delete p;
}
```

### Theory

**Constructor and Destructor Order**

- Local objects are constructed in declaration order.
- Destructors execute in reverse order.
- A block-local object is destroyed when its block ends.
- A dynamically allocated object is destroyed when `delete` is used.

**delete vs delete[]**

- `delete` is used for a single dynamically allocated object.
- `delete[]` is used for dynamically allocated arrays.
- Using the wrong form results in undefined behaviour.

**Destructor Rules**

- A destructor takes no arguments.
- A destructor cannot be overloaded.
- A class has only one destructor.

### RAII

RAII means Resource Acquisition Is Initialization.

- Resources are tied to an object's lifetime.
- When an object is destroyed, its resources are released automatically.
- This helps prevent memory leaks, including when exceptions occur.

### Predict the Output – Logger

**My Prediction:**

```
+A
+B
-B
use
copyAc
-Ac
+P
-P
end
-A
```

**Actual Output:**

```
+A
+B
-B
copyAc
use
-Ac
+P
-P
end
-A
```

### Explanation

1. Object `a` is created.
2. Object `b` is created and destroyed at the end of its block.
3. `use(a)` passes the object by value, so the copy constructor is called.
4. The copied object is destroyed after the function call.
5. Pointer object `p` is created dynamically.
6. `delete p` invokes its destructor.
7. At the end of `main`, object `a` is destroyed.

### Alternative Designs

- A single memory block using `new char[rows * cols]` requires one allocation instead of allocating each row separately.
- Access can be calculated using `r * cols + c`.
- This reduces pointer overhead.
- `vector<vector<char>>` is a safer alternative because it manages memory automatically.

---

## Q4 – Money – Operator Overloading

### Program

```
#include <iomanip>
#include <iostream>
using namespace std;

class Money {
    long long paise;

public:
    Money(long long rupees = 0, int p = 0)
        : paise(rupees * 100 + p) {}

    Money operator+(const Money& o) const {
        Money r;
        r.paise = paise + o.paise;
        return r;
    }

    Money operator-(const Money& o) const {
        Money r;
        r.paise = paise - o.paise;
        return r;
    }

    Money& operator+=(const Money& o) {
        paise += o.paise;
        return *this;
    }

    bool operator==(const Money& o) const {
        return paise == o.paise;
    }

    bool operator<(const Money& o) const {
        return paise < o.paise;
    }

    friend ostream& operator<<(ostream& out, const Money& m) {
        out << "Rs " << m.paise / 100 << '.'
            << setw(2) << setfill('0')
            << llabs(m.paise % 100);

        return out;
    }
};

int main() {
    Money a(120, 75), b(79, 25);

    Money c = a + b;

    cout << c << "\n";
    cout << boolalpha << (c == Money(200)) << "\n";

    Money d = Money(0, 75) + Money(0, 50);
    cout << d << "\n";
}
```

### Expected Output

```
Rs 200.00
true
Rs 1.25
```

### Theory

- `+` returns a new object and does not change the original operands.
- `+=` modifies the current object and returns a reference to `*this`.
- The `<<` operator must be a non-member function because its left operand is `ostream`, not `Money`.
- `a + b` internally calls `a.operator+(b)` when implemented as a member function.
- A member operator allows expressions such as `m + 100` when suitable conversions exist, but may not support `100 + m`.
- A non-member operator can support both operand orders when the conversions and overloads permit them.
- `Money _ Money` does not have a meaningful operator-overloading purpose.

### Why Use Integer Paise?

Floating-point calculations can produce precision errors.

For example:

```
0.1 + 0.2 != 0.3
```

Using `long long` to store paise avoids binary floating-point representation errors for these currency calculations.

Each addition takes O(1) time.

---

## Q5 – Playlist – Prefix and Postfix Operator Overloading

### Program

```
#include <iostream>
#include <vector>
#include <string>
using namespace std;

struct Time {
    int h, m, s;

    Time(int hh = 0, int mm = 0, int ss = 0)
        : h(hh), m(mm), s(ss) {}

    Time& operator++() {
        s++;

        if (s == 60) {
            s = 0;
            m++;
        }

        if (m == 60) {
            m = 0;
            h++;
        }

        return *this;
    }

    Time operator++(int) {
        Time old = *this;
        ++(*this);
        return old;
    }

    bool operator==(const Time& o) const {
        return h == o.h && m == o.m && s == o.s;
    }

    bool operator<(const Time& o) const {
        return h * 3600 + m * 60 + s <
               o.h * 3600 + o.m * 60 + o.s;
    }

    void show() {
        cout << h << ":" << m << ":" << s << "\n";
    }
};

class Playlist {
    vector<string> v;

public:
    Playlist(initializer_list<string> l) : v(l) {}

    string& operator[](int i) {
        if (i < 0 || i >= (int)v.size()) {
            static string err = "INVALID";
            cout << "Out of range\n";
            return err;
        }

        return v[i];
    }

    const string& operator[](int i) const {
        return v[i];
    }

    int size() const {
        return v.size();
    }
};

int main() {
    Time t(0, 59, 59);

    ++t;
    t.show();

    Time a = t++;
    cout << "a=";
    a.show();
}
```

### Expected Output

```
1:0:0
a=1:0:0
```

### Predict Output – Counter

```
Counter c{5};
a = c++;
b = ++c;
```

**Explanation:**

- Initially, `c = 5`.
- `c++` returns the old value 5 and then changes `c` to 6.
- `++c` increments `c` to 7 and returns the updated value.

**Output:**

```
5 7 7
```

### Theory

**Prefix Increment**

- Prefix increment uses `operator++()`.
- It increments the object and returns a reference to the updated object.

**Postfix Increment**

- Postfix increment uses `operator++(int)`.
- The dummy integer distinguishes it from prefix increment.
- It returns a copy of the old value.

**Why Prefer `++i`?**

For class types, postfix increment may create a temporary copy. Prefix increment avoids that extra copy when the old value is not needed.

**Const and Non-Const `operator[]`**

- A non-const object can use the non-const version to modify elements.
- A const object can use only the const version.
- Checked indexing adds an extra comparison, while the operation remains O(1).

---

## Q6 – Text Class – Copy Constructor and Operator Overloading

### Program

```
#include <iostream>
#include <cstring>
using namespace std;

class Text {
    char* s;
    int len;

public:
    Text(const char* p = "") {
        len = strlen(p);
        s = new char[len + 1];
        strcpy(s, p);
    }

    Text(const Text& o)
        : len(o.len), s(new char[o.len + 1]) {
        strcpy(s, o.s);
    }

    Text& operator=(const Text& o) {
        if (this == &o)
            return *this;

        delete[] s;

        len = o.len;
        s = new char[len + 1];

        strcpy(s, o.s);

        return *this;
    }

    ~Text() {
        delete[] s;
    }

    Text operator+(const Text& o) const {
        Text r;

        delete[] r.s;

        r.len = len + o.len;
        r.s = new char[r.len + 1];

        strcpy(r.s, s);
        strcat(r.s, o.s);

        return r;
    }

    bool operator==(const Text& o) const {
        return strcmp(s, o.s) == 0;
    }

    bool operator!=(const Text& o) const {
        return !(*this == o);
    }

    friend ostream& operator<<(ostream& out, const Text& t) {
        out << t.s;
        return out;
    }
};

int main() {
    Text a("Hello ");
    Text b("World");
    Text c("!");

    Text d = a + b + c;

    cout << d << "\n";
    cout << (d == Text("Hello World!")) << "\n";

    Text e;
    e = e;
}
```

### Expected Output

```
Hello World!
1
```

### Rules of Operator Overloading

The following operators cannot be overloaded:

- `::`
- `.`
- `.*`
- `?:`
- `sizeof`
- `typeid`

Other important rules:

- Operator precedence cannot be changed.
- Operator arity cannot be changed.
- At least one operand must be a user-defined type.
- The assignment operator `=` must be overloaded as a member function.
- A friend function is useful when the left operand is not the class type, such as `cout << t`, or when private access is needed.

### Performance Explanation

For the expression:

```
a + b + c
```

- Two temporary objects are created.
- String copying takes time proportional to the string lengths.
- An `a += b` style operation can reduce unnecessary temporary copies.
- `std::string` generally provides better memory management and optimized operations.

---

## Q7 – Complex Number – Operator Overloading

### Program

```
#include <iostream>
#include <cmath>
using namespace std;

class Complex {
    double re, im;

public:
    Complex(double r = 0, double i = 0)
        : re(r), im(i) {}

    Complex(double r) : re(r), im(0) {}

    Complex operator+(const Complex& o) const {
        return {re + o.re, im + o.im};
    }

    Complex operator-(const Complex& o) const {
        return {re - o.re, im - o.im};
    }

    Complex operator*(const Complex& o) const {
        return {
            re * o.re - im * o.im,
            re * o.im + im * o.re
        };
    }

    Complex operator/(const Complex& o) const {
        double d = o.re * o.re + o.im * o.im;

        if (d == 0) {
            cout << "Division by zero!\n";
            return {0, 0};
        }

        return {
            (re * o.re + im * o.im) / d,
            (im * o.re - re * o.im) / d
        };
    }

    bool operator==(const Complex& o) const {
        return re == o.re && im == o.im;
    }

    double mag() const {
        return sqrt(re * re + im * im);
    }

    friend ostream& operator<<(ostream& out, const Complex& c) {
        out << c.re << (c.im >= 0 ? "+" : "")
            << c.im << "j";

        return out;
    }
};

int main() {
    // Series: R=10, L: j*20, C: -j*5 => Z = 10+15j

    Complex R(10), L(0, 20), C(0, -5);

    Complex series = R + L + C;

    cout << "Series: " << series
         << " |Z|=" << series.mag() << "\n";

    // Parallel: Z1=10, Z2=10j => (10*10j)/(10+10j) = 5+5j

    Complex Z1(10), Z2(0, 10);

    Complex par = (Z1 * Z2) / (Z1 + Z2);

    cout << "Parallel: " << par
         << " |Z|=" << par.mag() << "\n";

    cout << "Div0 test: " << Z1 / Complex(0, 0) << "\n";
}
```

### Expected Output

```
Series: 10+15j |Z|=18.0278
Parallel: 5+5j |Z|=7.07107
Div0 test: Division by zero!
0+0j
```

### Hand Verification

**Series Circuit**

Given:

- Resistance = 10
- Inductive reactance = 20j
- Capacitive reactance = -5j

Therefore:

```
Z = 10 + 20j - 5j
Z = 10 + 15j
```

Magnitude:

```
|Z| = sqrt(10*10 + 15*15)
|Z| = 18.03
```

**Parallel Circuit**

Given:

```
Z1 = 10
Z2 = 10j
```

Formula:

```
Z = (Z1 * Z2) / (Z1 + Z2)
```

Result:

```
Z = 5 + 5j
```

Magnitude:

```
|Z| = sqrt(5*5 + 5*5)
|Z| = 7.07
```

### Comparison with `std::complex`

- Both use O(1) arithmetic operations.
- `std::complex` provides built-in complex number operations.
- It provides functions such as `abs()` and `arg()`.
- The custom `Complex` class demonstrates operator overloading and custom division-by-zero handling.
- The custom class is useful for understanding how operators work with user-defined types.

---

## Conclusion

This assignment demonstrates the use of constructors, destructors, dynamic memory allocation, copy constructors, and operator overloading in C++.

It explains the Rule of Three, deep copying, RAII, static and dynamic memory management, and the importance of safe resource handling.

The programs also demonstrate how operators such as `+`, `-`, `+=`, `++`, `[]`, `==`, and `<<` can be overloaded to make user-defined classes easier to use.

---

## Technologies Used

- **Programming Language:** C++
- **Standard:** C++17
- **Compiler:** GCC / G++
- **Concepts:** Constructors, Destructors, Copy Constructor, Dynamic Memory Allocation, Rule of Three, RAII, Operator Overloading
- **Libraries:** iostream, string, stdexcept, vector, iomanip, cstring, cmath
