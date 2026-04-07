# 1. Segmentation fault

*пример:* (segmantation_fault.cpp)

int main() {
	int* a = nullptr;
	*a = 5;

	return 0;
}

*статический анализ:* (clang-tidy)

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

*динамичсекий анализ:* (visual studio)

C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\x64\Debug\segmentation_fault.exe (процесс 25316) завершил работу с кодом -1073741819 (0xc0000005).

*причина:*

Разыменовывание нулевого указателя. Указатель a содержит нулевой адрес, который
недоступен для чтения/записи, поэтому при попытке записи по адресу *a = 5 программа 
завершается с ошибкой Segmentation fault.

# 2. Утечка памяти

*пример:* (memor_leak.cpp)

#define _CRTDBG_MAP_ALLOC
#include <stdlib.h>
#include <crtdbg.h>

int main() {
	_CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);

	int* a = new int[10];

	return 0;
}

*статический анализ:* (clang-tidy)

PS C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\memory_leak> clang-tidy memory_leak.cpp --checks='clang-analyzer-*' -- -std=c++11
2 warnings generated.
C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\memory_leak\memory_leak.cpp:2:7: warning: Value stored to 'a' during its initialization is never read [clang-analyzer-deadcode.DeadStores]
    2 |         int* a = new int[10];
      |              ^   ~~~~~~~~~~~
C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\memory_leak\memory_leak.cpp:2:7: note: Value stored to 'a' during its initialization is never read
    2 |         int* a = new int[10];
      |              ^   ~~~~~~~~~~~
C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\memory_leak\memory_leak.cpp:4:2: warning: Potential leak of memory pointed to by 'a' [clang-analyzer-cplusplus.NewDeleteLeaks]
    4 |         return 0;
      |         ^
C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\memory_leak\memory_leak.cpp:2:11: note: Memory is allocated
    2 |         int* a = new int[10];
      |                  ^~~~~~~~~~~
C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\memory_leak\memory_leak.cpp:4:2: note: Potential leak of memory pointed to by 'a'
    4 |         return 0;
      |         ^

*динамичсекий анализ:* (visual studio)

Detected memory leaks!
Dumping objects ->
{86} normal block at 0x00000148407EC000, 40 bytes long.
 Data: <                > CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD 
Object dump complete.

*причина:*

Отсутствие вызова delete для new. При выходе завершении программы указатель уничтожается,
и программа больше не может освободить эту память.

# 3. Ошибка линковки 

*пример:* (linker.cpp)

void print_number(int a);

int main() {
	print_number(5);

	return 0;
}

*статический анализ:*

не обнаружит ошибку, т.к. анализ происходит без выполнения кода, а ошибки при 
линковке выявляются после того, как исходный код уже преобразон в объектный файл

*динамичсекий анализ:* (visual studio)

код: LNK1120 описание: неразрешенных внешних элементов: 1

код: LNK2019 описание: ссылка на неразрешенный внешний символ "void __cdecl print_number(int)" (?print_number@@YAXH@Z) в функции main.

*причина:*

Функция объявлена, но не определена. После успешной компиляции, на этапе линковки компилятор
не смог найти реализацию функции, к которой обращаются.

# 4. Undefined behaviour 

*пример:* (undefined_behaviour.cpp)

int main() {
	int arr[5] = { 1, 2, 3, 4, 5 };
	int x = arr[10];

	return 0;
}

*статический анализ:* (clang-tidy)

PS C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\undefined_behaviour> clang-tidy undefined_behaviour.cpp --checks='clang-analyzer-core.*' -- -std=c++11
2 warnings generated.
C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\undefined_behaviour\undefined_behaviour.cpp:3:2: warning: Assigned value is uninitialized [clang-analyzer-core.uninitialized.Assign]
    3 |         int x = arr[10];
      |         ^       ~~~~~~~
C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\undefined_behaviour\undefined_behaviour.cpp:3:2: note: Assigned value is uninitialized
    3 |         int x = arr[10];
      |         ^       ~~~~~~~
C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\undefined_behaviour\undefined_behaviour.cpp:3:10: warning: array index 10 is past the end of the array (that has type 'int[5]') [clang-diagnostic-array-bounds]
    3 |         int x = arr[10];
      |                 ^   ~~
C:\Users\krist\OneDrive\Рабочий стол\cpp\lab4_po\undefined_behaviour\undefined_behaviour.cpp:2:2: note: array 'arr' declared here
    2 |         int arr[5] = { 1, 2, 3, 4, 5 };
      |         ^


*динамичсекий анализ:* (AddressSanitizer(visual studio))

==22564==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x00083eaff668
READ of size 4 at 0x00083eaff668 thread T0
    #0 0x7ff7deec1300 in main undefined_behaviour.cpp:3

Address 0x00083eaff668 is located in stack of thread T0 at offset 72 in frame
  This frame has 1 object(s):
    [32, 52) 'arr' <== Memory access at offset 72 overflows this variable

SUMMARY: AddressSanitizer: stack-buffer-overflow in main

*причина:*

Неопределенное поведение возникает при попытке записать в память за пределами
выделенной области. 