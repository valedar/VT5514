# Компиляция Socket Extension (sm-ext-socket) под Ubuntu 24.04

## Результат
Полностью статически слинкованное расширение `socket.ext.so` (~2.5 MB), работающее на любом 32-битном SourceMod сервере **без установки дополнительных библиотек**.

# Шаг 1: Установка системных зависимостей
sudo apt update
sudo apt install -y build-essential gcc-multilib g++-multilib wget git

# Шаг 2: Скачивание исходных кодов
## 2.1 SourceMod SDK (ВАЖНО: флаг --recursive)
cd ~
git clone --recursive https://github.com/alliedmodders/sourcemod
cd sourcemod
git checkout 1.12-dev

## 2.2 Metamod:Source SDK
cd ~
git clone https://github.com/alliedmodders/metamod-source
cd metamod-source
git checkout 1.12-dev

## 2.3 Расширение sm-ext-socket
cd ~
git clone https://github.com/nefarius/sm-ext-socket

# Шаг 3: Сборка 32-битных статических библиотек Boost
cd ~
wget https://archives.boost.io/release/1.74.0/source/boost_1_74_0.tar.gz
tar -xzf boost_1_74_0.tar.gz
cd boost_1_74_0
./bootstrap.sh --with-libraries=thread,system
./b2 address-model=32 link=static stage

# Шаг 4: Подготовка AMTL заголовков (ОБЯЗАТЕЛЬНО)
cd ~/sourcemod/public
ln -s amtl/amtl amtl_files

# Шаг 5: Редактирование Makefile
cd ~/sm-ext-socket
nano Makefile

## 5.1 Указываем пути к SDK (замените l4d2server на ваше имя пользователя)
SMSDK = /home/l4d2server/sourcemod
SOURCEMM = /home/l4d2server/metamod-source

## 5.2 Замена LINK
LINK = -lpthread -Wl,-Bstatic -static-libgcc -lboost_thread -lboost_system -lstdc++ -Wl,-Bdynamic

на этот:
LINK = -lpthread \
       -Wl,-Bstatic \
       -L/home/l4d2server/boost_1_74_0/stage/lib \
       -lboost_thread -lboost_system \
       -lstdc++ -lgcc_eh \
       -Wl,-Bdynamic \
       -m32 -shared

## 5.3 Заменить INCLUDE:
INCLUDE = -I. -I$(SOURCEMM) -I$(SOURCEMM)/sourcehook -I$(SOURCEMM)/sourcemm \
	-I$(SMSDK)/public -I$(SMSDK)/public/sourcepawn -I$(SMSDK)/public/extensions

на этот (ПОЛНАЯ СТАТИЧЕСКАЯ СБОРКА):
INCLUDE = -I. -I$(SOURCEMM)/public -I$(SOURCEMM)/public/sourcehook \
        -I$(SMSDK)/public -I$(SMSDK)/public/amtl \
        -I$(SMSDK)/public/amtl_files \
        -I$(SMSDK)/sourcepawn/include -I$(SMSDK)/public/extensions

## 5.4 Заменить CFLAGS:
CFLAGS = -D_LINUX -DSOURCEMOD_BUILD -Wall -fPIC -m32

на этот (должен быть -m32 ОБЯЗАТЕЛЬНО):
CFLAGS = -D_LINUX -DSOURCEMOD_BUILD -DBOOST_BIND_GLOBAL_PLACEHOLDERS -Wall -fPIC -m32

# Шаг 6: Исправление кода под Boost 1.74 (ОБЯЗАТЕЛЬНО)
cd ~/sm-ext-socket
cp Socket.cpp Socket.cpp.bak
sed -i 's/boost::system::posix_error::make_error_code(boost::system::posix_error::success)/boost::system::error_code()/g' Socket.cpp

## 6.1 Проверка: (должно быть пусто)
grep -n "posix_error" Socket.cpp

# Шаг 7: Финальная компиляция
cd ~/sm-ext-socket
make clean
make

## Ожидаемый вывод: (предупреждения о Boost bind — нормально)
gcc Release/Socket.ox ... -oRelease/socket.ext.so
make[1]: Leaving directory '/home/l4d2server/sm-ext-socket'

# Шаг 8: Проверка результата
ls -la Release/socket.ext.so
# Должно быть 2609044 байт где-то

file Release/socket.ext.so
# Должно быть: ELF 32-bit LSB shared object

ldd Release/socket.ext.so
# Должно быть ТОЛЬКО:
# linux-gate.so.1
# libc.so.6
# /lib/ld-linux.so.2

у меня так показывает:
linux-gate.so.1 (0xf5113000)
libc.so.6 => /usr/lib/i386-linux-gnu/libc.so.6 (0xf4cca000)
/lib/ld-linux.so.2 (0xf5115000)

# Шаг 9: Установка на сервер (где left4dead2 заменить на вашу игровую папку)
cp Release/socket.ext.so /путь/к/серверу/left4dead2/addons/sourcemod/extensions/

# Шаг 10: Проверка на сервере
Перезапустите сервер или в консоли выполните:

sm exts reload
sm exts list

Ожидаемый результат (пример):
[04] Socket (3.0.2): Socket extension for SourceMod
