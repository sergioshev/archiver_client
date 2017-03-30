TEMPLATE_TARGETS_EXEC=header_generator exim-init
TEMPLATE_TARGETS_COMON=exim.conf

TEMPLATE_TARGETS = $(TEMPLATE_TARGETS_COMON) $(TEMPLATE_TARGETS_EXEC)

ARCHIVER_DOMAIN_DATA = archiving_domain_data

NORMAL_TARGETS=exim4

include ../Makefile.rules

.PHONY: install
install : all
	$(foreach dir,$(ARCH_DIR) $(EXIM_SPOOL_DIR) \
		$(EXIM_LOG_DIR) $(EXIM_CONF_DIR) \
		$(EXIM_LIBS),install \
		-o $(ARCH_CLI_SYSTEM_USER) -m 755 -d $(dir) &&)true
	install -m 644 $(TEMPLATE_TARGETS_COMON) $(EXIM_CONF_DIR)
	install -m 640 -o $(ARCH_CLI_SYSTEM_USER) \
		$(ARCHIVER_DOMAIN_DATA) $(EXIM_CONF_DIR)
	install exim-lib/* $(EXIM_LIBS)
	install -m 755 $(TEMPLATE_TARGETS_EXEC) $(EXIM_CONF_DIR)
	cp -p $(NORMAL_TARGETS) $(EXIM_CONF_DIR)


