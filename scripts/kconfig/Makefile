# SPDX-License-Identifier: GPL-2.0
# ===========================================================================
# Kernel configuration targets
# These targets are used from top-level makefile

PHONY += xconfig gconfig menuconfig config silentoldconfig update-po-config \
	localmodconfig localyesconfig

ifdef KBUILD_KCONFIG
Kconfig := $(KBUILD_KCONFIG)
else
Kconfig := Kconfig
endif

ifeq ($(quiet),silent_)
silent := -s
endif

# We need this, in case the user has it in its environment
unexport CONFIG_

xconfig: $(obj)/qconf
	$< $(silent) $(Kconfig)

gconfig: $(obj)/gconf
	$< $(silent) $(Kconfig)

menuconfig: $(obj)/mconf
	$< $(silent) $(Kconfig)

config: $(obj)/conf
	$< $(silent) --oldaskconfig $(Kconfig)

nconfig: $(obj)/nconf
	$< $(silent) $(Kconfig)

silentoldconfig: $(obj)/conf
	$(Q)mkdir -p include/config include/generated
	$(Q)test -e include/generated/autoksyms.h || \
	    touch   include/generated/autoksyms.h
	$< $(silent) --$@ $(Kconfig)

localyesconfig localmodconfig: $(obj)/streamline_config.pl $(obj)/conf
	$(Q)mkdir -p include/config include/generated
	$(Q)perl $< --$@ $(srctree) $(Kconfig) > .tmp.config
	$(Q)if [ -f .config ]; then 					\
			cmp -s .tmp.config .config ||			\
			(mv -f .config .config.old.1;			\
			 mv -f .tmp.config .config;			\
			 $(obj)/conf $(silent) --silentoldconfig $(Kconfig); \
			 mv -f .config.old.1 .config.old)		\
	else								\
			mv -f .tmp.config .config;			\
			$(obj)/conf $(silent) --silentoldconfig $(Kconfig); \
	fi
	$(Q)rm -f .tmp.config

# Create new linux.pot file
# Adjust charset to UTF-8 in .po file to accept UTF-8 in Kconfig files
update-po-config: $(obj)/kxgettext $(obj)/gconf.glade.h
	$(Q)$(kecho) "  GEN     config.pot"
	$(Q)xgettext --default-domain=linux                         \
	    --add-comments --keyword=_ --keyword=N_                 \
	    --from-code=UTF-8                                       \
	    --files-from=$(srctree)/scripts/kconfig/POTFILES.in     \
	    --directory=$(srctree) --directory=$(objtree)           \
	    --output $(obj)/config.pot
	$(Q)sed -i s/CHARSET/UTF-8/ $(obj)/config.pot
	$(Q)(for i in `ls $(srctree)/arch/*/Kconfig      \
	    $(srctree)/arch/*/um/Kconfig`;               \
	    do                                           \
		$(kecho) "  GEN     $$i";                    \
		$(obj)/kxgettext $$i                     \
		     >> $(obj)/config.pot;               \
	    done )
	$(Q)$(kecho) "  GEN     linux.pot"
	$(Q)msguniq --sort-by-file --to-code=UTF-8 $(obj)/config.pot \
	    --output $(obj)/linux.pot
	$(Q)rm -f $(obj)/config.pot

# These targets map 1:1 to the commandline options of 'conf'
simple-targets := oldconfig allnoconfig allyesconfig allmodconfig \
	alldefconfig randconfig listnewconfig olddefconfig
PHONY += $(simple-targets)

$(simple-targets): $(obj)/conf
	$< $(silent) --$@ $(Kconfig)

PHONY += oldnoconfig savedefconfig defconfig

# oldnoconfig is an alias of olddefconfig, because people already are dependent
# on its behavior (sets new symbols to their default value but not 'n') with the
# counter-intuitive name.
oldnoconfig: olddefconfig

savedefconfig: $(obj)/conf
	$< $(silent) --$@=defconfig $(Kconfig)

defconfig: $(obj)/conf
ifeq ($(KBUILD_DEFCONFIG),)
	$< $(silent) --defconfig $(Kconfig)
else
ifneq ($(wildcard $(srctree)/arch/$(SRCARCH)/configs/$(KBUILD_DEFCONFIG)),)
	@$(kecho) "*** Default configuration is based on '$(KBUILD_DEFCONFIG)'"
	$(Q)$< $(silent) --defconfig=arch/$(SRCARCH)/configs/$(KBUILD_DEFCONFIG) $(Kconfig)
else
	@$(kecho) "*** Default configuration is based on target '$(KBUILD_DEFCONFIG)'"
	$(Q)$(MAKE) -f $(srctree)/Makefile $(KBUILD_DEFCONFIG)
endif
endif

%_defconfig: $(obj)/conf
	$(Q)$< $(silent) --defconfig=arch/$(SRCARCH)/configs/$@ $(Kconfig)

configfiles=$(wildcard $(srctree)/kernel/configs/$@ $(srctree)/arch/$(SRCARCH)/configs/$@)

%.config: $(obj)/conf
	$(if $(call configfiles),, $(error No configuration exists for this target on this architecture))
	$(Q)$(CONFIG_SHELL) $(srctree)/scripts/kconfig/merge_config.sh -m .config $(configfiles)
	+$(Q)yes "" | $(MAKE) -f $(srctree)/Makefile oldconfig

PHONY += kvmconfig
kvmconfig: kvm_guest.config
	@:

PHONY += xenconfig
xenconfig: xen.config
	@:

PHONY += tinyconfig
tinyconfig:
	$(Q)$(MAKE) -f $(srctree)/Makefile allnoconfig tiny.config

# Help text used by make help
help:
	@echo  '  config	  - Update current config utilising a line-oriented program'
	@echo  '  nconfig         - Update current config utilising a ncurses menu based'
	@echo  '                    program'
	@echo  '  menuconfig	  - Update current config utilising a menu based program'
	@echo  '  xconfig	  - Update current config utilising a Qt based front-end'
	@echo  '  gconfig	  - Update current config utilising a GTK+ based front-end'
	@echo  '  oldconfig	  - Update current config utilising a provided .config as base'
	@echo  '  localmodconfig  - Update current config disabling modules not loaded'
	@echo  '  localyesconfig  - Update current config converting local mods to core'
	@echo  '  silentoldconfig - Same as oldconfig, but quietly, additionally update deps'
	@echo  '  defconfig	  - New config with default from ARCH supplied defconfig'
	@echo  '  savedefconfig   - Save current config as ./defconfig (minimal config)'
	@echo  '  allnoconfig	  - New config where all options are answered with no'
	@echo  '  allyesconfig	  - New config where all options are accepted with yes'
	@echo  '  allmodconfig	  - New config selecting modules when possible'
	@echo  '  alldefconfig    - New config with all symbols set to default'
	@echo  '  randconfig	  - New config with random answer to all options'
	@echo  '  listnewconfig   - List new options'
	@echo  '  olddefconfig	  - Same as silentoldconfig but sets new symbols to their'
	@echo  '                    default value'
	@echo  '  kvmconfig	  - Enable additional options for kvm guest kernel support'
	@echo  '  xenconfig       - Enable additional options for xen dom0 and guest kernel support'
	@echo  '  tinyconfig	  - Configure the tiniest possible kernel'

# ===========================================================================
# Shared Makefile for the various kconfig executables:
# conf:	  Used for defconfig, oldconfig and related targets
# object files used by all kconfig flavours

conf-objs	:= conf.o  zconf.tab.o
kxgettext-objs	:= kxgettext.o zconf.tab.o

hostprogs-y := conf kxgettext

targets		+= zconf.lex.c
clean-files	+= gconf.glade.h
clean-files     += config.pot linux.pot

# Add environment specific flags
HOST_EXTRACFLAGS += $(shell $(CONFIG_SHELL) $(srctree)/$(src)/check.sh $(HOSTCC) $(HOSTCFLAGS)) \
		    -DLOCALE

# generated files seem to need this to find local include files
HOSTCFLAGS_zconf.lex.o	:= -I$(src)
HOSTCFLAGS_zconf.tab.o	:= -I$(src)

LEX_PREFIX_zconf	:= zconf
YACC_PREFIX_zconf	:= zconf

# nconf: Used for the nconfig target based on ncurses
hostprogs-y	+= nconf
nconf-objs	:= nconf.o zconf.tab.o nconf.gui.o

HOSTLOADLIBES_nconf	= $(shell . $(obj)/.nconf-cfg && echo $$libs)
HOSTCFLAGS_nconf.o	= $(shell . $(obj)/.nconf-cfg && echo $$cflags)
HOSTCFLAGS_nconf.gui.o	= $(shell . $(obj)/.nconf-cfg && echo $$cflags)

$(obj)/nconf.o: $(obj)/.nconf-cfg

# mconf: Used for the menuconfig target based on lxdialog
hostprogs-y	+= mconf
lxdialog	:= checklist.o inputbox.o menubox.o textbox.o util.o yesno.o
mconf-objs	:= mconf.o zconf.tab.o $(addprefix lxdialog/, $(lxdialog))

HOSTLOADLIBES_mconf = $(shell . $(obj)/.mconf-cfg && echo $$libs)
$(foreach f, mconf.o $(lxdialog), \
  $(eval HOSTCFLAGS_$f = $$(shell . $(obj)/.mconf-cfg && echo $$$$cflags)))

$(addprefix $(obj)/, mconf.o $(lxdialog)): $(obj)/.mconf-cfg

# qconf: Used for the xconfig target based on Qt
hostprogs-y	+= qconf
qconf-cxxobjs	:= qconf.o
qconf-objs	:= zconf.tab.o

HOSTLOADLIBES_qconf	= $(shell . $(obj)/.qconf-cfg && echo $$libs)
HOSTCXXFLAGS_qconf.o	= $(shell . $(obj)/.qconf-cfg && echo $$cflags)

$(obj)/qconf.o: $(obj)/.qconf-cfg $(obj)/qconf.moc

quiet_cmd_moc = MOC     $@
      cmd_moc = $(shell . $(obj)/.qconf-cfg && echo $$moc) -i $< -o $@

$(obj)/%.moc: $(src)/%.h $(obj)/.qconf-cfg
	$(call cmd,moc)

# gconf: Used for the gconfig target based on GTK+
hostprogs-y	+= gconf
gconf-objs	:= gconf.o zconf.tab.o

HOSTLOADLIBES_gconf = $(shell . $(obj)/.gconf-cfg && echo $$libs)
HOSTCFLAGS_gconf.o  = $(shell . $(obj)/.gconf-cfg && echo $$cflags)

$(obj)/gconf.o: $(obj)/.gconf-cfg

$(obj)/zconf.tab.o: $(obj)/zconf.lex.c

# Extract gconf menu items for i18n support
$(obj)/gconf.glade.h: $(obj)/gconf.glade
	$(Q)intltool-extract --type=gettext/glade --srcdir=$(srctree) \
	$(obj)/gconf.glade

# check if necessary packages are available, and configure build flags
define filechk_conf_cfg
	$(CONFIG_SHELL) $<
endef

$(obj)/.%conf-cfg: $(src)/%conf-cfg.sh FORCE
	$(call filechk,conf_cfg)

clean-files += .*conf-cfg
