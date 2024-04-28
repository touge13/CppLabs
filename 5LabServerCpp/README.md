Completed by: `Grudinin Mikhail Artemovich (Грудинин Михаил Артемович)`

# Простой многопоточный сервер

Этот проект реализует простой многопоточный сервер для обработки HTTP-запросов. Сервер позволяет клиентам загружать, просматривать, редактировать и удалять файлы на сервере через веб-интерфейс. Данный сервер может служить каркасом для приложения "Notes".

## Контекст запросов

Сервер предоставляет следующие возможности:
- Просмотр списка файлов на сервере.
- Загрузка текстовых файлов на сервер.
- Просмотр, редактирование и скачивание текстовых файлов.
- Удаление файлов с сервера.

## Архитектура приложения

Архитектура приложения следующая:
- **Функция `main` вызывает функцию `server`, которая:** Отвечает за создание, привязку и прослушивание серверного сокета. В бесконечном цикле принимает входящие соединения и обрабатывает каждое соединение в отдельном потоке.
- **Функция `handleClient` для обработки клиентских запросов:** Отвечает за разбор и обработку HTTP-запросов от клиентов. Эта функция выполняют различные действия в зависимости от метода запроса и запрошенного пути.
- **Функции для работы с файлами:** Отвечают за редактирование (`editFile`), загрузки (`uploadFile`), просмотр содержимого файла (`sendFile`), список всех файлов (`sendFileList`) и удаления (`deleteFile`) файлов на сервере.
- **Функции `logger`, `sendErrorResponse`, `urlDecode`:** Отвечают за логирование (логи записываются в файл build/server.log), отправку клиенту HTTP-ответа с информацией об ошибке и декодирования URL-кодированной строки.

Реализованы `POST` и `GET` запросы.

`POST`:
- Загрузка файла на сервер.
- Удаление файла с сервера.
- Изменение файла (загрузка в файл измененное содержимое).

`GET`:
- Просмотр содержимого файла.
- Просмотр списка файлов.
- Запрос на изменение файла (просмотр содержимого до изменения).

## Структура приложения

Структура приложения следующая:
- **Папка `build`** содержит реализацию сборки проекта при помощи CMake и server.log.
- **Папка `files`** содержит файлы с .txt расширением.
- **Папка `include`** содержит .h файлы.
- **Папка `src`** содержит .cpp файлы.

## Технологии и инструменты

Для реализации сервера используются следующие технологии и инструменты:
- Язык программирования C++.
- Библиотеки языка C++:
    1. **iostream**: для потокового ввода-вывода. 
    2. **sys/socket.h**: для работы с сокетами. 
    3. **netinet/in.h**: для работы с сетевыми адресами и протоколами.
    4. **unistd.h**: для стандартных функций операционной системы, такие как `close()`. 
    5. **string**: для работы со строками.
    6. **thread**: для создания и управления потоками.
    7. **mutex**: для обеспечения безопасности при доступе к разделяемым ресурсам из нескольких потоков.
    8. **vector**: для работы с динамическими массивами.
    9. **fstream**: для работы с файлами. 
    10. **dirent.h**: для работы с каталогами и файлами в них. 
    11. **iomanip**: для форматирования ввода-вывода.
    12. **sstream**: для работы с потоками строк. 
    13. **ctime**: для работы со временем.
    14. **semaphore.h**: для ограничения максимального количества потоков.
    15. **atomic**: для более гладкого завершения работы приложения.
- POSIX-совместимые функции для работы с сокетами (например, функции `socket`, `bind`, `listen`, `accept`, которые являются частью `POSIX API`).
- Многопоточность с использованием `std::thread` и мьютексов (`std::mutex`):
    - **Потоки (`std::thread`):** Создаются для обработки каждого клиентского запроса в отдельном потоке, обеспечивая параллельную обработку запросов. Иными словами, сервер будет прослушивать определенный порт и ожидать входящие запросы от клиентов. Когда клиент подключается, создается новый поток для обработки запроса.
    - **Синхронизация потоков (`std::mutex`)** Мьютексы используются для обеспечения синхронизации потоков и избежания гонок между ними.

    _`std::mutex` используется в функции `deleteFile` для блокировки доступа к стандартному выводу, пока текущий поток не закончит свою операцию. В многопоточной среде доступ к ресурсам, которые могут изменяться из разных потоков, должен быть синхронизирован. Это необходимо для обеспечения безопасности в многопоточной среде. Использование мьютекса позволяет избежать проблем с совместным доступом к стандартному выводу из разных потоков, гарантируя последовательность вывода и корректное отображение информации._

   _В функциях `sendFile`, `sendFileList`, `editFile` и `uploadFile` не изменяются общие данные напрямую, а лишь отправляют HTTP-ответы клиенту. Поскольку отправка ответов в сокет является операцией вывода, которая не вызывается одновременно из нескольких потоков (поскольку каждый сокет-клиент обрабатывается в отдельном потоке), блокировка мьютекса в этих функциях не является необходимой. Поэтому добавление блокировки мьютекса в этих функциях в данном случае не требуется._

- Semaphore:
    _Чтобы добавить ограничение на количество потоков, которые могут обрабатывать клиентов одновременно, используем семафор. Семафор представляет собой счетчик, который уменьшается при создании нового потока и увеличивается при завершении работы потока. Если счетчик достигнет максимального значения (мы его сами устанавливаем), новые потоки не будут создаваться до тех пор, пока не завершится работа какого-либо из текущих потоков._

- Освобождение потоков:
    _Потоки освобождаются после завершения обработки запроса от пользователя. В цикле `while (true)` потоки создаются для обработки каждого входящего соединения. После создания потока для обработки клиентского запроса вызывается `threads.emplace_back(handleClient, clientSocket);`. Этот вызов добавляет новый поток в вектор потоков threads, который затем обрабатывает запрос клиента в функции handleClient._
    _После завершения обработки запроса в функции handleClient, поток завершается, и объект потока удаляется из вектора. Когда объект потока удаляется, он автоматически освобождает свои ресурсы, включая использованную память и другие ресурсы (Когда поток завершает выполнение функции, он возвращает управление обратно в основной цикл в функции server, и его ресурсы освобождаются автоматически)._

- Сигнал потоку о его завершении:
    _Чтобы отправить сигнал потоку о его завершении перед вызовом `accept`, можно использовать механизм флагов синхронизации. Добавим флаг, который будет сигнализировать потокам о необходимости завершения работы, и проверять его перед вызовом `accept`._

    _Если флаг `shouldTerminate` установлен, поток просто завершает свою работу, не принимая новых соединений. Для корректного завершения работы всех потоков после остановки сервера используется функция `join()`._

    _Отправка сигнала потоку о его завершении перед вызовом `accept` может быть необходима, если требуется прервать ожидание нового соединения в случае завершения работы сервера или при необходимости выключить сервер_
    _Например, если наш сервер работает в бесконечном цикле, ожидая входящих соединений, и нам нужно остановить его, отправка сигнала потокам о завершении работы позволит им корректно завершить выполнение, закрыть открытые сокеты и освободить другие ресурсы перед завершением программы._
    _Это беспечивает более гладкое завершение работы приложения и позволяет избежать утечек ресурсов или других нежелательных последствий._

## Google Code Style

Код программы соответствует [Google Code Style](https://google.github.io/styleguide/cppguide.html).

## Запуск сервера

Чтобы запустить сервер, скомпилируйте и запустите исполняемый файл. По умолчанию сервер прослушивает порт 8080. Вы можете изменить порт в соответствии с вашими требованиями в файле server.cpp, за это отвечает переменная port.

Проект собран при помощи CMake (требует установки на ваш ПК):

build/CMakeLists.txt:
```
cmake_minimum_required(VERSION 3.13)
project(visualizationGraph)
set(CMAKE_CXX_STANDARD 11)

include_directories("include/")

add_executable(main 
    ../src/main.cpp
    ../src/deleteFile.cpp
    ../src/editFile.cpp
    ../src/handleClient.cpp
    ../src/logger.cpp
    ../src/sendErrorResponse.cpp
    ../src/sendFile.cpp
    ../src/sendFileList.cpp
    ../src/server.cpp
    ../src/uploadFile.cpp
    ../src/urlDecode.cpp
)
```
Сборка при помощи CMake:
```
mkdir build
cd build
cmake build .
make 
```
Запуск:
```
./main
```

## Нагрузочное тестирование
Для реализации нагрузочного тестирования сервера воспользуемся ApacheBench (ab):

1. Установим ab
Для macOS:```brew install ab```

2. Запустим тестирование 
```ab -n 100 -c 100 http://127.0.0.1:8080/```
Параметры -n и -c в утилите ApacheBench (ab) означают следующее:
-n указывает общее количество запросов, которые следует выполнить.
-c указывает количество одновременных запросов (конкурентных соединений).

3. Смотрим на результат тестирования
```
This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 127.0.0.1 (be patient).....done


Server Software:        /h1><p><a
Server Hostname:        127.0.0.1
Server Port:            8080

Document Path:          /
Document Length:        1705 bytes

Concurrency Level:      100
Time taken for tests:   0.013 seconds
Complete requests:      100
Failed requests:        0
Total transferred:      174600 bytes
HTML transferred:       170500 bytes
Requests per second:    7992.97 [#/sec] (mean)
Time per request:       12.511 [ms] (mean)
Time per request:       0.125 [ms] (mean, across all concurrent requests)
Transfer rate:          13628.63 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    4   1.3      5       6
Processing:     2    3   1.0      4       7
Waiting:        0    3   0.9      3       5
Total:          6    8   1.0      8      11

Percentage of the requests served within a certain time (ms)
  50%      8
  66%      8
  75%      8
  80%      8
  90%      9
  95%      9
  98%     11
  99%     11
 100%     11 (longest request)
```

Результаты теста показывают, что сервер успешно обрабатывает 100 запросов с 100 одновременными соединениями. Ни один запрос не завершился неудачей (Failed requests: 0).
Некоторые ключевые метрики:
- **Requests per second (RPS)**: Среднее количество запросов в секунду, обрабатываемых сервером. В нашем случае это примерно 7992.97 запроса в секунду.
- **Time per request**: Время, затраченное на обработку каждого запроса. Среднее время составляет около 12.511 миллисекунд.
- **Transfer rate**: Скорость передачи данных от сервера к клиенту, выраженная в килобайтах в секунду. Наш сервер передал данные со скоростью около 13628.63 килобайт в секунду.