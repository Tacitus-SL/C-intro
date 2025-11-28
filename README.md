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

Часто импортируемые библиотеки в заданиях:
```c
#include <stdio.h>      // стандартный ввод/вывод: printf, scanf, fprintf, fopen
#include <stdlib.h>     // общие утилиты: malloc, calloc, realloc, free, exit, atoi
#include <string.h>     // работа со строками: strcpy, strcat, strlen, strcmp, memset
#include <unistd.h>     // POSIX API: fork, exec, pipe, read, write, close, sleep, getpid
#include <errno.h>      // для работы с кодами ошибок errno
#include <fcntl.h>      // управление файлами и дескрипторами: open, O_RDONLY, O_CREAT
#include <sys/types.h>  // основные типы данных: pid_t, size_t, key_t
#include <sys/stat.h>   // структура stat, режимы файлов, mkdir
#include <time.h>       // работа с временем: time, clock, nanosleep
#include <signal.h>     // работа с сигналами: sigaction, signal, raise, kill
#include <pthread.h>    // POSIX потоки: pthread_create, pthread_mutex, pthread_cond, pthread_barrier
#include <semaphore.h>  // семафоры: sem_init, sem_wait, sem_post, sem_destroy
#include <stdatomic.h>  // атомарные операции: atomic_int, atomic_fetch_add
#include <sys/wait.h>   // работа с процессами: wait, waitpid, WNOHANG
#include <sys/mman.h>   // shared memory: mmap, munmap, PROT_READ/WRITE, MAP_SHARED
#include <sys/ipc.h>    // IPC ключи: ftok, IPC_CREAT
#include <sys/msg.h>    // очереди сообщений: msgget, msgsnd, msgrcv
#include <sys/sem.h>    // системные семафоры (SysV): semget, semop
#include <sys/shm.h>    // системная shared memory: shmget, shmat, shmdt
#include <sys/socket.h> // сокеты: socket, bind, listen, accept, connect
#include <netinet/in.h> // структуры для работы с IP (sockaddr_in)
#include <arpa/inet.h>  // преобразование IP адресов: inet_addr, inet_ntoa
#include <sys/select.h> // I/O Multiplexing: select, FD_SET, FD_ZERO
#include <sys/epoll.h>  // I/O Multiplexing (Linux): epoll_create, epoll_ctl, epoll_wait
#include <sys/syscall.h> // syscall для futex и других системных вызовов
```

---

## 2. Типы данных и модификаторы

### Базовые типы C

```c
int a = 10;
unsigned int b = 20;
long c = 1000000;
float f = 3.14;
double d = 2.71828;
char ch = 'A';
```

- `const` — переменная только для чтения.

- `volatile` — значение может меняться извне, компилятор не оптимизирует доступ.
  
---

## 3. Указатели и разыменование

- `*` — разыменование  
- `&` — получение адреса


```c
int x = 5;
int *p = &x;     // указатель на x
printf("%d\n", *p); // разыменование, вывод 5

int arr[3] = {1,2,3};
int *ptr = arr;  // имя массива — указатель на первый элемент
```

---

## 4. Структуры и указатели на них

```c
struct Point {
    int x, y;
};

struct Point p1 = {10, 20};
struct Point *pp = &p1;

printf("%d\n", pp->x); // 10
```
-> разыменовывает указатель на структуру и обращается к её полю в одном шаге.

## 5. Динамическая память: `malloc`, `calloc`, `realloc`, `free`

### 5.1 `malloc`
Выделяет непрерывный блок памяти.

```c
int *arr = (int*)malloc(5 * sizeof(int));
```

Память **не инициализирована**.

### 5.2 `calloc`
Выделяет и **обнуляет** память.

```c
int *arr = (int*)calloc(5, sizeof(int));
```

### 5.3 `realloc`
Меняет размер существующего блока.

```c
arr = (int*)realloc(arr, 10 * sizeof(int));
```

### 5.4 `free`
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

## 6. Zombie и Daemon процессы
### 6.1 Создание процессов

```c
#include <unistd.h>
#include <sys/wait.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    pid_t pid = fork();
    if (pid == 0) {
        printf("Child process\n");
        exit(0);
    } else {
        wait(NULL); // ждём дочерний процесс
        printf("Parent process\n");
    }
}
```

### 6.2 Zombie

Процесс завершён, но родитель не вызвал `wait()`, то есть все еще остается в системе, но в то же время не занимают CPU.

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

### 6.3 Daemon

Фоновый процесс, не привязанный к терминалу и не имеющий родителя4.

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

## 7. Сигналы
Сигнал — это уведомление ядра процессу.
Сигнал может:
- SIGKILL: прервать программу,
- SIGPIPE: завершить программу,
- SIGCHLD: ничего не сделать (если он игнорируется),
- SIGSTOP: остановить выполнения,
- SIGCONT: возобновить выполнение.
```c
#include <signal.h>
#include <stdio.h>

void handler(int sig) {          // Функция-обработчик сигнала
    printf("Caught %d\n", sig); // Выводим номер сигнала, который пришёл
}

int main() {
    struct sigaction sa;          // Создаём структуру, описывающую поведение сигнала

    sa.sa_handler = handler;      // Назначаем обработчик сигнала (нашу функцию handler)
    sigemptyset(&sa.sa_mask);     // Инициализируем маску сигналов — пока выполняется handler, другие сигналы не блокируются
    sa.sa_flags = 0;              // Флаги поведения сигнала (0 — базовое поведение без дополнительных опций)

    sigaction(SIGINT, &sa, NULL); // Привязываем структуру sa к сигналу SIGINT (Ctrl+C)
                                   // NULL — старое действие сигнала не сохраняем

    while(1);                     
}

```

---

## 7. Многопоточность и синхронизация

### 7.1 Создание потока
```c
#include <pthread.h>
#include <stdio.h>

void* thread_func(void* arg) {
    printf("Hello from thread %d\n", *(int*)arg);
    return NULL;
}

int main() {
    pthread_t t;
    int id = 1;
    pthread_create(&t, NULL, thread_func, &id);
    pthread_join(t, NULL);
}
```

### 7.2 Mutex
Mutex — это механизм, который позволяет только одному потоку одновременно работать с ресурсом, предотвращая гонку данных.
```c
pthread_mutex_t lock;
pthread_mutex_init(&lock, NULL);

pthread_mutex_lock(&lock);
// критическая секция
pthread_mutex_unlock(&lock);
```

### 7.2 Semaphore
Semaphore — это счётчик, который ограничивает одновременный доступ потоков или процессов к ресурсу
```c
#include <semaphore.h>
sem_t sem;
sem_init(&sem, 0, 3); // 3 ресурса

sem_wait(&sem); // захватываем
sem_post(&sem); // освобождаем

```

### 7.3 Condition Variable
Condition variable — это механизм, который позволяет потоку ждать определённого события и синхронизироваться с другими потоками через сигнал.
```c
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

pthread_mutex_lock(&mutex);
pthread_cond_wait(&cond, &mutex); // ждём события
pthread_mutex_unlock(&mutex);

pthread_cond_signal(&cond); // сигнал для пробуждения

```

### 7.4 Barrier
Barrier — это точка синхронизации, где несколько потоков ждут друг друга, прежде чем все смогут продолжить выполнение.
```c
pthread_barrier_t barrier;
pthread_barrier_init(&barrier, NULL, 3); // ждём 3 потока
pthread_barrier_wait(&barrier); // каждый поток ждёт остальных
```

### 7.5 Atomic operations
Atomic operation — это операция над переменной, которая выполняется полностью без прерываний, гарантируя корректный доступ в многопоточном окружении.
```c
#include <stdatomic.h>
atomic_int a = 0;
atomic_fetch_add(&a, 1);
```

### 7.6 Таблица
| Механизм             | Основная задача                  | Ограничения / Состояние             | Особенности использования |
|----------------------|---------------------------------|------------------------------------|--------------------------|
| Mutex                | Взаимное исключение, защита критической секции | Только один поток за раз           | Захватывает ресурс, другие ждут; подходит для защиты общих данных |
| Semaphore            | Ограничение количества потоков/ресурсов       | Счётчик ≥ 0                        | Позволяет N потокам одновременно работать с ресурсом; бинарный или счётный |
| Condition Variable   | Ждать наступления события                      | Не хранит состояния                | Используется с mutex; поток ждёт события, освобождая mutex пока ждёт |
| Barrier              | Синхронизация всех потоков на точке            | Ждёт указанное количество потоков  | Все потоки останавливаются на барьере, пока все не достигнут; потом продолжают работу |
| Atomic Operations    | Безопасная работа с простыми переменными      | Неразрывные операции               | Выполняются полностью без прерывания; подходят для lock-free счётчиков и флагов |

---

## 8. IPC (межпроцессное взаимодействие)

### 8.1 Pipes
```c
int fd[2];
pipe(fd); // создаём канал
write(fd[1], "Hi", 2);
char buf[3];
read(fd[0], buf, 2);
```

### 8.2 Shared memory
```c
#include <sys/shm.h>
int id = shmget(IPC_PRIVATE, 1024, 0666|IPC_CREAT);
char *mem = shmat(id, NULL, 0);
```

---

## 9. I/O Multiplexing

Позволяет работать с несколькими файловыми дескрипторами.

Используемые функции:

```
select()
poll()
epoll()
```
