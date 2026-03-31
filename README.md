# 🔌 Компиляция Socket Extension (sm-ext-socket) под Ubuntu 24.04

Полностью статически слинкованное расширение `socket.ext.so`, работающее
на любом 32-битном SourceMod сервере.

------------------------------------------------------------------------

## 📦 Шаг 1: Установка зависимостей

``` bash
sudo apt update
sudo apt install -y build-essential gcc-multilib g++-multilib wget git
```

------------------------------------------------------------------------

## 📥 Шаг 2: Скачивание исходников

### 2.1 SourceMod SDK (ВАЖНО: `--recursive`)

``` bash
cd ~
git clone --recursive https://github.com/alliedmodders/sourcemod
cd sourcemod
git checkout 1.12-dev
```

### 2.2 Metamod:Source SDK

``` bash
cd ~
git clone https://github.com/alliedmodders/metamod-source
cd metamod-source
git checkout 1.12-dev
```

### 2.3 sm-ext-socket

``` bash
cd ~
git clone https://github.com/nefarius/sm-ext-socket
```

------------------------------------------------------------------------

## 🧱 Шаг 3: Сборка Boost (32-bit, static)

``` bash
cd ~
wget https://archives.boost.io/release/1.74.0/source/boost_1_74_0.tar.gz
tar -xzf boost_1_74_0.tar.gz
cd boost_1_74_0

./bootstrap.sh --with-libraries=thread,system
./b2 address-model=32 link=static stage
```

------------------------------------------------------------------------

## 🧩 Шаг 4: Подготовка AMTL (ОБЯЗАТЕЛЬНО)

``` bash
cd ~/sourcemod/public
ln -s amtl/amtl amtl_files
```

------------------------------------------------------------------------

## ⚙️ Шаг 5: Настройка Makefile

``` bash
cd ~/sm-ext-socket
nano Makefile
```

### 5.1 Указать пути к SDK (путь правильный делайте, USERNAME -- это пример)

``` make
SMSDK = /home/USERNAME/sourcemod
SOURCEMM = /home/USERNAME/metamod-source
```

### 5.2 Заменить LINK на этот (путь правильный делайте, USERNAME -- это пример)

``` make
LINK = -lpthread \
       -Wl,-Bstatic \
       -L/home/USERNAME/boost_1_74_0/stage/lib \
       -lboost_thread -lboost_system \
       -lstdc++ -lgcc_eh \
       -Wl,-Bdynamic \
       -m32 -shared
```

### 5.3 Заменить INCLUDE на этот

``` make
INCLUDE = -I. -I$(SOURCEMM)/public -I$(SOURCEMM)/public/sourcehook \
        -I$(SMSDK)/public -I$(SMSDK)/public/amtl \
        -I$(SMSDK)/public/amtl_files \
        -I$(SMSDK)/sourcepawn/include -I$(SMSDK)/public/extensions
```

### 5.4 Заменить CFLAGS на этот

``` make
CFLAGS = -D_LINUX -DSOURCEMOD_BUILD -DBOOST_BIND_GLOBAL_PLACEHOLDERS -Wall -fPIC -m32
```

------------------------------------------------------------------------

## 🛠 Шаг 6: Исправление под Boost 1.74

``` bash
cd ~/sm-ext-socket
cp Socket.cpp Socket.cpp.bak
sed -i 's/boost::system::posix_error::make_error_code(boost::system::posix_error::success)/boost::system::error_code()/g' Socket.cpp
```

### Проверка

``` bash
grep -n "posix_error" Socket.cpp
```

------------------------------------------------------------------------

## 🧪 Шаг 7: Компиляция

``` bash
cd ~/sm-ext-socket
make clean
make
```

Ожидаемый вывод:

``` bash
gcc Release/Socket.ox ... -oRelease/socket.ext.so
make[1]: Leaving directory '/home/USENAME/sm-ext-socket'
```

------------------------------------------------------------------------

## 🔍 Шаг 8: Проверка результата

``` bash
ls -la Release/socket.ext.so
file Release/socket.ext.so
ldd Release/socket.ext.so
```

Ожидаемо:

``` bash
2609044 байт(где-то)
```

``` bash
ELF 32-bit LSB shared object
```

``` bash
linux-gate.so.1
libc.so.6
/lib/ld-linux.so.2
```
------------------------------------------------------------------------

## 🚀 Шаг 9: Установка

``` bash
cp Release/socket.ext.so /путь/к/серверу/left4dead2/addons/sourcemod/extensions/
```

------------------------------------------------------------------------

## ✅ Шаг 10: Проверка

``` bash
sm exts reload
sm exts list
```

Ожидаемый результат:

    [04] Socket (3.0.2): Socket extension for SourceMod

------------------------------------------------------------------------

## 💡 Примечания

-   Обязательно `-m32`
-   Boost только static
-   Фикс Boost обязателен
-   AMTL симлинк обязателен
