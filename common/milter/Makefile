LIBS_SENDMAIL=/usr/lib/libmilter/libmilter.a
LDFLAGS=-pthread
CFLAGS=
C=cc
MILTER_NAME=mailforward
OBJ_TARGETS=
TEMPLATE_TARGETS_EXEC=milter-init
TEMPLATE_TARGETS=$(TEMPLATE_TARGETS_EXEC)

include ../Makefile.rules

%.o : %.c
	$(CC) -c -o $@ $<

.PHONY: all
all: $(MILTER_NAME).c $(OBJ_TARGETS) $(TEMPLATE_TARGETS)
	$(CC) $(LDFLAGS) $(CFLAGS) -o $(MILTER_NAME) $< $(LIBS_SENDMAIL) $(OBJ_TARGETS)

.PHONY: clean
clean:
	rm -f *.o
	rm -f $(TEMPLATE_TARGETS_EXEC)

.PHONY: install
install: all
	install -d $(MILTER_DIR)
	install $(MILTER_NAME) $(MILTER_DIR)
	install -m 755 $(TEMPLATE_TARGETS_EXEC) $(MILTER_DIR)

