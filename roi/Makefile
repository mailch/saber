CC=g++
CPPFLAGS=-g -O2 -Wall -Wextra -Isrc -rdynamic -std=c++11 -DNDEBUG $(OPTFLAGS)
LDLIBS=-ldl -luv $(OPTLIBS)
PREFIX?=/usr/local

SOURCES=$(wildcard src/olist.cpp src/proto.cpp)
OBJECTS=$(patsubst %.cpp,%.o,$(SOURCES))

TEST_SRC=$(wildcard tests/*_tests.cpp)
TESTS=$(patsubst %.cpp,%,$(TEST_SRC))

SERVER_SRC=$(wildcard src/gserver.cpp)
SERVER=$(patsubst %.cpp,%,$(SERVER_SRC))

CLIENT_SRC=$(wildcard src/gclient.cpp)
CLIENT=$(patsubst %.cpp,%,$(CLIENT_SRC))

TARGET=build/libroi.a
SO_TARGET=$(patsubst %.a,%.so,$(TARGET))

all: $(TARGET) $(SO_TARGET) server client tests

dev: CFLAGS=-g -Wall -Isrc -Wall -Wextra $(OPTFLAGS)
dev: all

$(TARGET): CPPFLAGS += -fPIC
$(TARGET): build $(OBJECTS)
	ar rcs $@ $(OBJECTS)
	ranlib $@

$(SO_TARGET): $(TARGET) $(OBJECTS)
	$(CC) -shared -o $@ $(OBJECTS)

build:
	@mkdir -p build
	@mkdir -p bin

.PHONY: tests
tests: LDLIBS += $(TARGET)
tests: $(TESTS)
	sh ./tests/runtests.sh

.PHONY: server
server: LDLIBS += $(TARGET)
server: $(SERVER)

.PHONY: client
client: LDLIBS += $(TARGET)
client: $(CLIENT)

valgrind:
	VALGRIND="valgrind --log-file=/tmp/valgrind-%p.log" $(MAKE)

clean:
	rm -rf build $(OBJECTS) $(TESTS) $(SERVER) $(CLIENT)
	rm -f tests/tests.log
	find . -name "*.gc*" -exec rm {} \;
	rm -rf `find . -name "*.dSYM" -print`

install: all
	install -d $(DESTDIR)/$(PREFIX)/lib/
	install $(TARGET) $(DESTDIR)/$(PREFIX)/lib/

BADFUNC='[^_.>a-zA-Z0-9](str(n?cpy|n?cat|xfrm|n?dup|str|pbrk|tok|_)|stpn?cpy|a?sn?printf|byte_)'
check:
	@echo Files with potentially dangerous functions.
	@egrep $(BADFUNC) $(SOURCES) || true
