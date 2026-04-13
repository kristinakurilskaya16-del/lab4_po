# 1. Segmentation fault

**Пример:** (`segmentation_fault.cpp`)

    int main() {
        int* a = nullptr;
        *a = 5;
        return 0;
    }

**Статический анализ:** (clang-tidy)

    PS C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\segmentation_fault> clang-tidy segmentation_fault.cpp --checks='clang-analyzer-*' -- -std=c++11
    1 warning generated.
    C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\segmentation_fault\segmentation_fault.cpp:3:5: warning: Dereference of null pointer (loaded from variable 'a') [clang-analyzer-core.NullDereference]
        3 |         *a = 5;
          |          ~ ^
    C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\segmentation_fault\segmentation_fault.cpp:2:2: note: 'a' initialized to a null pointer value
        2 |         int* a = nullptr;
          |         ^~~~~~
    C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\segmentation_fault\segmentation_fault.cpp:3:5: note: Dereference of null pointer (loaded from variable 'a')
        3 |         *a = 5;
          |          ~ ^

**Динамический анализ:** (Visual Studio)

    C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\x64\Debug\segmentation_fault.exe (процесс 25316) завершил работу с кодом -1073741819 (0xc0000005).

**Причина:**

Разыменовывание нулевого указателя. Указатель `a` содержит нулевой адрес, который недоступен для чтения/записи, поэтому при попытке записи по адресу `*a = 5` программа завершается с ошибкой Segmentation fault.

---

# 2. Утечка памяти

**Пример:** (`memory_leak.cpp`)

    #define _CRTDBG_MAP_ALLOC
    #include <stdlib.h>
    #include <crtdbg.h>
    
    int main() {
        _CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);
        int* a = new int[10];
        return 0;
    }

**Статический анализ:** (clang-tidy)

    PS C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\memory_leak> clang-tidy memory_leak.cpp --checks='clang-analyzer-*' -- -std=c++11
    2 warnings generated.
    C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\memory_leak\memory_leak.cpp:2:7: warning: Value stored to 'a' during its initialization is never read [clang-analyzer-deadcode.DeadStores]
        2 |         int* a = new int[10];
          |              ^   ~~~~~~~~~~~
    C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\memory_leak\memory_leak.cpp:4:2: warning: Potential leak of memory pointed to by 'a' [clang-analyzer-cplusplus.NewDeleteLeaks]
        4 |         return 0;
          |         ^

**Динамический анализ:** (Visual Studio)

    Detected memory leaks!
    Dumping objects ->
    {86} normal block at 0x00000148407EC000, 40 bytes long.
     Data: <                > CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD 
    Object dump complete.

**Причина:**

Отсутствие вызова `delete[]` для `new[]`. При завершении программы указатель уничтожается, и программа больше не может освободить эту память.

---

# 3. Ошибка линковки

**Пример:** (`linker.cpp`)

    void print_number(int a);
    
    int main() {
        print_number(5);
        return 0;
    }

**Статический анализ:**

Не обнаружит ошибку, т.к. анализ происходит без выполнения кода, а ошибки при линковке выявляются после того, как исходный код уже преобразован в объектный файл.

**Динамический анализ:** (Visual Studio)

    код: LNK1120 описание: неразрешенных внешних элементов: 1
    код: LNK2019 описание: ссылка на неразрешенный внешний символ "void __cdecl print_number(int)" (?print_number@@YAXH@Z) в функции main.

**Причина:**

Функция объявлена, но не определена. После успешной компиляции на этапе линковки компилятор не смог найти реализацию функции, к которой обращаются.

---

# 4. Undefined behaviour

**Пример:** (`undefined_behaviour.cpp`)

    int main() {
        int arr[5] = { 1, 2, 3, 4, 5 };
        int x = arr[10];
        return 0;
    }

**Статический анализ:** (clang-tidy)

    PS C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\undefined_behaviour> clang-tidy undefined_behaviour.cpp --checks='clang-analyzer-core.*' -- -std=c++11
    2 warnings generated.
    C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\undefined_behaviour\undefined_behaviour.cpp:3:2: warning: Assigned value is uninitialized [clang-analyzer-core.uninitialized.Assign]
        3 |         int x = arr[10];
          |         ^       ~~~~~~~
    C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\undefined_behaviour\undefined_behaviour.cpp:3:10: warning: array index 10 is past the end of the array (that has type 'int[5]') [clang-diagnostic-array-bounds]
        3 |         int x = arr[10];
          |                 ^   ~~

**Динамический анализ:** (AddressSanitizer в Visual Studio)

    ==22564==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x00083eaff668
    READ of size 4 at 0x00083eaff668 thread T0
        #0 0x7ff7deec1300 in main undefined_behaviour.cpp:3
    Address 0x00083eaff668 is located in stack of thread T0 at offset 72 in frame
      [32, 52) 'arr' <== Memory access at offset 72 overflows this variable
    SUMMARY: AddressSanitizer: stack-buffer-overflow in main

**Причина:**

Неопределенное поведение возникает при попытке чтения памяти за пределами выделенной области (выход за границы массива).
