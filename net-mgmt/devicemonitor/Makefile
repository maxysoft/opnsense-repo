PLUGIN_NAME=		devicemonitor
PLUGIN_VERSION=		2.3
PLUGIN_REVISION=	1
PLUGIN_COMMENT=		Device Monitor - network device tracking
PLUGIN_MAINTAINER=	maxysoft
PLUGIN_WWW=		https://github.com/maxysoft/opnsense-devicemonitor

# No PLUGIN_DEPENDS on purpose. The plugin needs python3 and its sqlite3
# module, and both are guaranteed: the opnsense meta package depends on
# py313-sqlite3, which pulls in python313. Depending on "python3" would resolve
# against the FreeBSD ports repo at build time but fail on a firewall, because
# the OPNsense repository has no package by that name.

.include "../../Mk/plugins.mk"
