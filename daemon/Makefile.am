## Process this file with automake to produce Makefile.in

## Copyright (C) 2005-2011 Red Hat, Inc.
## See COPYING.LIB for the License of this software

INCLUDES = \
	-I$(top_srcdir)/gnulib/lib -I../gnulib/lib \
	-I$(top_srcdir)/include -I$(top_builddir)/include \
	-I$(top_srcdir)/src -I../src \
	-I$(top_srcdir)/src/util \
	-I$(top_srcdir)/src/conf \
	-I$(top_srcdir)/src/rpc \
	-I$(top_srcdir)/src/remote \
	$(GETTEXT_CPPFLAGS)

CLEANFILES =

DAEMON_GENERATED =					\
		$(srcdir)/remote_dispatch.h		\
		$(srcdir)/qemu_dispatch.h

DAEMON_SOURCES =					\
		libvirtd.c libvirtd.h			\
		remote.c remote.h			\
		stream.c stream.h			\
		../src/remote/remote_protocol.c		\
		../src/remote/qemu_protocol.c		\
		$(DAEMON_GENERATED)

DISTCLEANFILES =
EXTRA_DIST =						\
	remote_dispatch.h				\
	qemu_dispatch.h					\
	libvirtd.conf					\
	libvirtd.init.in				\
	libvirtd.upstart				\
	libvirtd.policy-0				\
	libvirtd.policy-1				\
	libvirtd.sasl					\
	libvirtd.sysconf				\
	libvirtd.sysctl					\
	libvirtd.aug                                    \
	libvirtd.logrotate.in                           \
	libvirtd.qemu.logrotate.in                      \
	libvirtd.lxc.logrotate.in                       \
	libvirtd.uml.logrotate.in                       \
	test_libvirtd.aug                               \
	THREADS.txt					\
	libvirtd.pod.in					\
	libvirtd.8.in					\
	$(DAEMON_SOURCES)

BUILT_SOURCES =

REMOTE_PROTOCOL = $(top_srcdir)/src/remote/remote_protocol.x
QEMU_PROTOCOL = $(top_srcdir)/src/remote/qemu_protocol.x

$(srcdir)/remote_dispatch.h: $(srcdir)/../src/rpc/gendispatch.pl \
		$(REMOTE_PROTOCOL)
	$(AM_V_GEN)perl -w $(srcdir)/../src/rpc/gendispatch.pl -b remote \
	  $(REMOTE_PROTOCOL) > $@

$(srcdir)/qemu_dispatch.h: $(srcdir)/../src/rpc/gendispatch.pl \
		$(QEMU_PROTOCOL)
	$(AM_V_GEN)perl -w $(srcdir)/../src/rpc/gendispatch.pl -b qemu \
	  $(QEMU_PROTOCOL) > $@

if WITH_LIBVIRTD

man8_MANS = libvirtd.8

sbin_PROGRAMS = libvirtd

confdir = $(sysconfdir)/libvirt/
conf_DATA = libvirtd.conf

augeasdir = $(datadir)/augeas/lenses
augeas_DATA = libvirtd.aug

augeastestsdir = $(datadir)/augeas/lenses/tests
augeastests_DATA = test_libvirtd.aug

libvirtd.8: $(srcdir)/libvirtd.8.in
	sed \
	    -e 's![@]sysconfdir[@]!$(sysconfdir)!g' \
	    -e 's![@]localstatedir[@]!$(localstatedir)!g' \
	    -e 's![@]remote_pid_file[@]!$(REMOTE_PID_FILE)!g' \
	    < $< > $@-t
	mv $@-t $@

libvirtd_SOURCES = $(DAEMON_SOURCES)

#-D_XOPEN_SOURCE=600 -D_XOPEN_SOURCE_EXTENDED=1 -D_POSIX_C_SOURCE=199506L
libvirtd_CFLAGS = \
	$(LIBXML_CFLAGS) $(GNUTLS_CFLAGS) $(SASL_CFLAGS) \
	$(XDR_CFLAGS) $(POLKIT_CFLAGS) \
	$(WARN_CFLAGS) \
	$(COVERAGE_CFLAGS) \
	-DQEMUD_PID_FILE="\"$(QEMUD_PID_FILE)\"" \
	-DREMOTE_PID_FILE="\"$(REMOTE_PID_FILE)\""

libvirtd_LDFLAGS =					\
	$(WARN_CFLAGS)					\
	$(COVERAGE_LDFLAGS)

libvirtd_LDADD =					\
	$(LIBXML_LIBS)					\
	$(GNUTLS_LIBS)					\
	$(SASL_LIBS)					\
	$(POLKIT_LIBS)

if WITH_DTRACE
libvirtd_LDADD += ../src/probes.o
endif

libvirtd_LDADD += \
	../src/libvirt-qemu.la

if ! WITH_DRIVER_MODULES
if WITH_QEMU
    libvirtd_LDADD += ../src/libvirt_driver_qemu.la
endif

if WITH_LXC
    libvirtd_LDADD += ../src/libvirt_driver_lxc.la
endif

if WITH_LIBXL
    libvirtd_LDADD += ../src/libvirt_driver_libxl.la
endif

if WITH_UML
    libvirtd_LDADD += ../src/libvirt_driver_uml.la
endif

if WITH_STORAGE_DIR
    libvirtd_LDADD += ../src/libvirt_driver_storage.la
endif

if WITH_NETWORK
    libvirtd_LDADD += ../src/libvirt_driver_network.la
endif

if WITH_NETCF
    libvirtd_LDADD += ../src/libvirt_driver_interface.la
endif

if WITH_NODE_DEVICES
    libvirtd_LDADD += ../src/libvirt_driver_nodedev.la
endif

if WITH_SECRETS
    libvirtd_LDADD += ../src/libvirt_driver_secret.la
endif

if WITH_NWFILTER
    libvirtd_LDADD += ../src/libvirt_driver_nwfilter.la
endif
endif

libvirtd_LDADD += ../src/libvirt.la

if HAVE_POLKIT
if HAVE_POLKIT0
policydir = $(datadir)/PolicyKit/policy
policyfile = libvirtd.policy-0
else
policydir = $(datadir)/polkit-1/actions
policyfile = libvirtd.policy-1
endif
endif

install-data-local: install-init install-data-sasl install-data-polkit \
		    install-logrotate
	mkdir -p $(DESTDIR)$(localstatedir)/log/libvirt
	mkdir -p $(DESTDIR)$(localstatedir)/run/libvirt
	mkdir -p $(DESTDIR)$(localstatedir)/lib/libvirt

uninstall-local:: uninstall-init uninstall-data-sasl uninstall-data-polkit
	rmdir $(DESTDIR)$(localstatedir)/log/libvirt || :
	rmdir $(DESTDIR)$(localstatedir)/run/libvirt || :
	rmdir $(DESTDIR)$(localstatedir)/lib/libvirt || :

if HAVE_POLKIT
install-data-polkit:: install-init
	mkdir -p $(DESTDIR)$(policydir)
	$(INSTALL_DATA) $(srcdir)/$(policyfile) $(DESTDIR)$(policydir)/org.libvirt.unix.policy
uninstall-data-polkit:: install-init
	rm -f $(DESTDIR)$(policydir)/org.libvirt.unix.policy
else
install-data-polkit::
uninstall-data-polkit::
endif

remote.c: $(DAEMON_GENERATED)
remote.h: $(DAEMON_GENERATED)

LOGROTATE_CONFS = libvirtd.qemu.logrotate libvirtd.lxc.logrotate \
		  libvirtd.uml.logrotate libvirtd.logrotate

BUILT_SOURCES += $(LOGROTATE_CONFS)

libvirtd.logrotate: libvirtd.logrotate.in
	sed							\
	    -e 's![@]localstatedir[@]!$(localstatedir)!g'	\
	    < $< > $@-t
	mv $@-t $@

libvirtd.qemu.logrotate: libvirtd.qemu.logrotate.in
	sed							\
	    -e 's![@]localstatedir[@]!$(localstatedir)!g'	\
	    < $< > $@-t
	mv $@-t $@

libvirtd.lxc.logrotate: libvirtd.lxc.logrotate.in
	$(AM_V_GEN)sed						\
	    -e 's![@]localstatedir[@]!$(localstatedir)!g'	\
	    < $< > $@-t &&					\
	    mv $@-t $@

libvirtd.uml.logrotate: libvirtd.uml.logrotate.in
	$(AM_V_GEN)sed						\
	    -e 's![@]localstatedir[@]!$(localstatedir)!g'	\
	    < $< > $@-t &&					\
	    mv $@-t $@

install-logrotate: $(LOGROTATE_CONFS)
	mkdir -p $(DESTDIR)$(localstatedir)/log/libvirt/qemu/
	mkdir -p $(DESTDIR)$(localstatedir)/log/libvirt/lxc/
	mkdir -p $(DESTDIR)$(localstatedir)/log/libvirt/uml/
	mkdir -p $(DESTDIR)$(sysconfdir)/logrotate.d/
	$(INSTALL_DATA) libvirtd.logrotate $(DESTDIR)$(sysconfdir)/logrotate.d/libvirtd
	$(INSTALL_DATA) libvirtd.qemu.logrotate $(DESTDIR)$(sysconfdir)/logrotate.d/libvirtd.qemu
	$(INSTALL_DATA) libvirtd.lxc.logrotate $(DESTDIR)$(sysconfdir)/logrotate.d/libvirtd.lxc
	$(INSTALL_DATA) libvirtd.uml.logrotate $(DESTDIR)$(sysconfdir)/logrotate.d/libvirtd.uml

if LIBVIRT_INIT_SCRIPT_RED_HAT
install-init: libvirtd.init
	mkdir -p $(DESTDIR)$(sysconfdir)/rc.d/init.d \
	  $(DESTDIR)$(sysconfdir)/sysconfig \
	  $(DESTDIR)$(sysconfdir)/sysctl.d
	$(INSTALL_SCRIPT) libvirtd.init \
	  $(DESTDIR)$(sysconfdir)/rc.d/init.d/libvirtd
	$(INSTALL_DATA) $(srcdir)/libvirtd.sysconf \
	  $(DESTDIR)$(sysconfdir)/sysconfig/libvirtd
	$(INSTALL_DATA) $(srcdir)/libvirtd.sysctl \
	  $(DESTDIR)$(sysconfdir)/sysctl.d/libvirtd

uninstall-init:
	rm -f $(DESTDIR)$(sysconfdir)/rc.d/init.d/libvirtd \
		$(DESTDIR)$(sysconfdir)/sysconfig/libvirtd \
		$(DESTDIR)$(sysconfdir)/sysctl.d/libvirtd

BUILT_SOURCES += libvirtd.init

libvirtd.init: libvirtd.init.in $(top_builddir)/config.status
	$(AM_V_GEN)sed					\
	    -e s!\@localstatedir\@!@localstatedir@!g	\
	    -e s!\@sbindir\@!@sbindir@!g		\
	    -e s!\@sysconfdir\@!@sysconfdir@!g		\
	    < $< > $@-t &&				\
	    chmod a+x $@-t &&				\
	    mv $@-t $@

check-local:
	$(AM_V_GEN)if test -x '$(AUGPARSE)'; then \
	  '$(AUGPARSE)' -I $(srcdir) $(srcdir)/test_libvirtd.aug; \
	fi

else

install-init:
uninstall-init:
libvirtd.init:

endif # LIBVIRT_INIT_SCRIPT_RED_HAT

# This must be added last, since functions it provides/replaces
# are used by nearly every other library.
libvirtd_LDADD += ../gnulib/lib/libgnu.la $(LIBSOCKET)

else # WITH_LIBVIRTD
install-data-local: install-data-sasl
uninstall-local:: uninstall-data-sasl
endif # WITH_LIBVIRTD

POD2MAN = pod2man -c "Virtualization Support" \
			-r "$(PACKAGE)-$(VERSION)" -s 8

$(srcdir)/libvirtd.8.in: libvirtd.pod.in
	$(AM_V_GEN)$(POD2MAN) $< $@

# This is needed for clients too, so can't wrap in
# the WITH_LIBVIRTD conditional
if HAVE_SASL
install-data-sasl:
	mkdir -p $(DESTDIR)$(sysconfdir)/sasl2/
	$(INSTALL_DATA) $(srcdir)/libvirtd.sasl $(DESTDIR)$(sysconfdir)/sasl2/libvirt.conf

uninstall-data-sasl:
	rm -f $(DESTDIR)$(sysconfdir)/sasl2/libvirt.conf
	rmdir $(DESTDIR)$(sysconfdir)/sasl2/
else
install-data-sasl:
uninstall-data-sasl:
endif


CLEANFILES += $(BUILT_SOURCES) $(man8_MANS)
CLEANFILES += *.cov *.gcov .libs/*.gcda .libs/*.gcno *.gcno *.gcda
MAINTAINERCLEANFILES = $(srcdir)/libvirtd.8.in $(DAEMON_GENERATED)
