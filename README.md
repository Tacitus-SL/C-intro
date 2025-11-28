# C / Linux: Шпаргалка

## 1. Препроцессор: `#include` и `#define`

`#include` вставляет содержимое другого файла **до компиляции**.

### Два способа:

#### `<file>` — ищет файл в системных директориях
```c
#include <stdio.h> // стандартная библиотека ввода/вывода
```

#### `"file"` — ищет сначала в текущей директории, потом в системной
```c
#include "myheader.h"
```

---

### `#define`

Создаёт макрос (текстовую подстановку).

#### Пример константы
```c
#define PI 3.14159
```

#### Пример функции-макроса
```c
#define SQUARE(x) ((x) * (x))
```

#### Разница с `const`
- `#define` — чистая текстовая подстановка, нет проверки типов.
- `const` — настоящая переменная:

```c
const double pi = 3.14159;
```

---

## 2. Динамическая память: `malloc`, `calloc`, `realloc`, `free`

### 2.1 `malloc`
Выделяет непрерывный блок памяти.

```c
int *arr = (int*)malloc(5 * sizeof(int));
```

Память **не инициализирована**.

### 2.2 `calloc`
Выделяет и **обнуляет** память.

```c
int *arr = (int*)calloc(5, sizeof(int));
```

### 2.3 `realloc`
Меняет размер существующего блока.

```c
arr = (int*)realloc(arr, 10 * sizeof(int));
```

### 2.4 `free`
Освобождает выделенную память.

```c
free(arr);
arr = NULL;
```

---

### Различия между `malloc`, `calloc`, `realloc`

| Функция | Инициализация | Аргументы | Назначение |
|--------|---------------|-----------|------------|
| malloc | нет | размер в байтах | выделяет память |
| calloc | да (0) | количество, размер одного | выделяет и обнуляет |
| realloc | сохраняет старые данные | указатель, новый размер | меняет размер блока |

---

## 3. Zombie и Daemon процессы

### 3.1 Zombie

Процесс завершён, но родитель не вызвал `wait()`.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();
    if (pid == 0) {
        printf("Child done\n");
        exit(0);
    } else {
        sleep(10); // дочерний — zombie
    }
}
```

**Как избежать:** вызывать `wait()` / `waitpid()`.

---

### 3.2 Daemon

Фоновый процесс, отсоединённый от терминала.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();
    if (pid > 0) exit(0);
    setsid();
    while(1) {
        sleep(1);
    }
}
```

---

## 4. Указатели и разыменование

- `*` — разыменование  
- `&` — получение адреса

```c
int x = 5;
int *p = &x;
printf("%d\n", *p); // 5
```

---

## 5. Структуры и указатели на них

```c
struct Point {
    int x, y;
};

struct Point p1 = {10, 20};
struct Point *pp = &p1;

printf("%d\n", pp->x); // 10
```

---

## 6. Многопоточность и синхронизация

### 6.1 Mutex
```c
pthread_mutex_t lock;
pthread_mutex_init(&lock, NULL);

pthread_mutex_lock(&lock);
// критическая секция
pthread_mutex_unlock(&lock);
```

### 6.2 Semaphore
```c
sem_t sem;
sem_init(&sem, 0, 3);

sem_wait(&sem);
sem_post(&sem);
```

### 6.3 Condition Variable
```c
pthread_cond_wait(&cond, &mutex);
pthread_cond_signal(&cond);
```

### 6.4 Barrier
Ожидает всех потоков на точке синхронизации.

### 6.5 Atomic operations
```c
#include <stdatomic.h>
atomic_int a = 0;
atomic_fetch_add(&a, 1);
```

### 6.6 Futex
Низкоуровневый механизм синхронизации Linux.

---

## 7. IPC (межпроцессное взаимодействие)

- Shared memory  
- Pipes  
- Message Queues  
- Sockets  

---

## 8. I/O Multiplexing

Позволяет работать с несколькими файловыми дескрипторами.

Используемые функции:

```
select()
poll()
epoll()
```
