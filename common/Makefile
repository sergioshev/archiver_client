SUBDIRS=exim milter
include Makefile.defs

all install: % : arch_starter
	$(foreach I,$(SUBDIRS),$(MAKE) -C $I $@ && ) true


clean : % : arch_clean
	$(foreach I,$(SUBDIRS),$(MAKE) -C $I $@ && ) true



.PHONY: arch_starter
arch_starter: arch-init.template Makefile.defs
	./instanciateMacros Makefile.defs  < arch-init.template > arch-init
	install -m 755 -d $(ARCH_DIR)
	install -m 755 arch-init $(ARCH_DIR)

.PHONY: arch_clean
arch_clean:
	rm -f arch-init

