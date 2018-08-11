
-include config.unix

datarootdir = $(prefix)/share
bindir = $(prefix)/bin
sysconfdir = $(prefix)/etc
INSTALL = install
INSTALL_DATA = $(INSTALL) -m 644
BINARY_PLATFORM = unix

SHEBANG = \#!$(LUA_BINDIR)/$(LUA_INTERPRETER)
LUA = $(LUA_BINDIR)/$(LUA_INTERPRETER)
luarocksconfdir = $(sysconfdir)/luarocks
luadir = $(datarootdir)/lua/$(LUA_VERSION)
builddir = ./build
buildbinarydir = ./build-binary


LUAROCKS_FILES = $(shell find src/luarocks/ -type f -name '*.lua')

all: build

# ----------------------------------------
# Base build
# ----------------------------------------

build: luarocks luarocks-admin $(builddir)/luarocks $(builddir)/luarocks-admin

config.unix:
	@echo Please run the "./configure" script before building.
	@echo
	@exit 1

$(builddir)/config-$(LUA_VERSION).lua: config.unix
	mkdir -p "$(@D)"
	@printf -- '-- LuaRocks configuration\n\n'\
	'rocks_trees = {\n'\
	'   { name = "user", root = home .. "/.luarocks" };\n'\
	'   { name = "system", root = "'"$(rocks_tree)"'" };\n'\
	'}\n'\
	"$$([ -n "$(LUA_INTERPRETER)" ] && printf 'lua_interpreter = "%s";\\n' "$(LUA_INTERPRETER)")"\
	'variables = {\n'\
	"$$([ -n "$(LUA_DIR)" ] && printf '   LUA_DIR = "%s";\\n' "$(LUA_DIR)")"\
	"$$([ -n "$(LUA_INCDIR)" ] && printf '   LUA_INCDIR = "%s";\\n' "$(LUA_INCDIR)")"\
	"$$([ -n "$(LUA_BINDIR)" ] && printf '   LUA_BINDIR = "%s";\\n' "$(LUA_BINDIR)")"\
	"$$([ -n "$(LUA_LIBDIR)" ] && printf '   LUA_LIBDIR = "%s";\\n' "$(LUA_LIBDIR)")"\
	'}\n'\
	> $@

luarocks: config.unix $(builddir)/config-$(LUA_VERSION).lua
	rm -f src/luarocks/core/hardcoded.lua
	echo "#!/bin/sh" > luarocks
	echo "unset LUA_PATH LUA_PATH_5_2 LUA_PATH_5_3 LUA_PATH_5_4" >> luarocks
	echo 'LUAROCKS_SYSCONFDIR="$(luarocksconfdir)" LUA_PATH="$(CURDIR)/src/?.lua;;" exec "$(LUA)" "$(CURDIR)/src/bin/luarocks" --project-tree="$(CURDIR)/lua_modules" "$$@"' >> luarocks
	chmod +rx ./luarocks
	./luarocks init
	cp $(builddir)/config-$(LUA_VERSION).lua .luarocks/config-$(LUA_VERSION).lua

luarocks-admin: config.unix
	rm -f src/luarocks/core/hardcoded.lua
	echo "#!/bin/sh" > luarocks-admin
	echo "unset LUA_PATH LUA_PATH_5_2 LUA_PATH_5_3 LUA_PATH_5_4" >> luarocks-admin
	echo 'LUAROCKS_SYSCONFDIR="$(luarocksconfdir)" LUA_PATH="$(CURDIR)/src/?.lua;;" exec "$(LUA)" "$(CURDIR)/src/bin/luarocks-admin" --project-tree="$(CURDIR)/lua_modules" "$$@"' >> luarocks-admin
	chmod +rx ./luarocks-admin

$(builddir)/luarocks: src/bin/luarocks config.unix
	mkdir -p "$(@D)"
	(printf '$(SHEBANG)\n'\
	'package.loaded["luarocks.core.hardcoded"] = { '\
	"$$([ -n "$(FORCE_CONFIG)" ] && printf 'FORCE_CONFIG = true, ')"\
	'SYSCONFDIR = [[$(luarocksconfdir)]] }\n'\
	'package.path=[[$(luadir)/?.lua;]] .. package.path\n'; \
	tail -n +2 src/bin/luarocks \
	)> "$@"

$(builddir)/luarocks-admin: src/bin/luarocks-admin config.unix
	mkdir -p "$(@D)"
	(printf '$(SHEBANG)\n'\
	'package.loaded["luarocks.core.hardcoded"] = { '\
	"$$([ -n "$(FORCE_CONFIG)" ] && printf 'FORCE_CONFIG = true, ')"\
	'SYSCONFDIR = [[$(luarocksconfdir)]] }\n'\
	'package.path=[[$(luadir)/?.lua;]] .. package.path\n'; \
	tail -n +2 src/bin/luarocks-admin \
	)> "$@"

# ----------------------------------------
# Base build
# ----------------------------------------

binary: luarocks $(buildbinarydir)/luarocks.exe $(buildbinarydir)/luarocks-admin.exe

$(buildbinarydir)/luarocks.exe: src/bin/luarocks $(LUAROCKS_FILES)
	(unset $(LUA_ENV_VARS); \
	"$(LUA)" binary/all_in_one "$<" "$(LUA_DIR)" "^src/luarocks/admin/" "$(luarocksconfdir)" "$(@D)" "$(FORCE_CONFIG)" $(BINARY_PLATFORM) $(CC) $(NM) $(SYSROOT))

$(buildbinarydir)/luarocks-admin.exe: src/bin/luarocks-admin $(LUAROCKS_FILES)
	(unset $(LUA_ENV_VARS); \
	"$(LUA)" binary/all_in_one "$<" "$(LUA_DIR)" "^src/luarocks/cmd/" "$(luarocksconfdir)" "$(@D)" "$(FORCE_CONFIG)" $(BINARY_PLATFORM) $(CC) $(NM) $(SYSROOT))

# ----------------------------------------
# Regular install
# ----------------------------------------

INSTALL_FILES = $(DESTDIR)$(bindir)/luarocks \
	$(DESTDIR)$(bindir)/luarocks-admin \
	$(DESTDIR)$(luarocksconfdir)/config-$(LUA_VERSION).lua \
	$(patsubst src/%, $(DESTDIR)$(luadir)/%, $(LUAROCKS_FILES))

install: $(INSTALL_FILES)

$(DESTDIR)$(bindir)/luarocks: $(builddir)/luarocks
	mkdir -p "$(@D)"
	$(INSTALL) "$<" "$@"

$(DESTDIR)$(bindir)/luarocks-admin: $(builddir)/luarocks-admin
	mkdir -p "$(@D)"
	$(INSTALL) "$<" "$@"

$(DESTDIR)$(luadir)/luarocks/%.lua: src/luarocks/%.lua
	mkdir -p "$(@D)"
	$(INSTALL_DATA) "$<" "$@"

$(DESTDIR)$(luarocksconfdir)/config-$(LUA_VERSION).lua: $(builddir)/config-$(LUA_VERSION).lua
	mkdir -p "$(@D)"
	$(INSTALL_DATA) "$<" "$@"

uninstall:
	rm -rf $(INSTALL_FILES)

# ----------------------------------------
# Binary install
# ----------------------------------------

LUAROCKS_CORE_FILES = $(wildcard src/luarocks/core/* src/luarocks/loader.lua)
INSTALL_BINARY_FILES = $(patsubst src/%, $(DESTDIR)$(luadir)/%, $(LUAROCKS_CORE_FILES)) \
	$(DESTDIR)$(luarocksconfdir)/config-$(LUA_VERSION).lua

install-binary: $(INSTALL_BINARY_FILES)
	mkdir -p "$(buildbinarydir)"
	$(INSTALL) "$(buildbinarydir)/luarocks.exe" "$(DESTDIR)$(bindir)/luarocks"
	$(INSTALL) "$(buildbinarydir)/luarocks-admin.exe" "$(DESTDIR)$(bindir)/luarocks-admin"

# ----------------------------------------
# Bootstrap install
# ----------------------------------------

bootstrap: luarocks $(DESTDIR)$(luarocksconfdir)/config-$(LUA_VERSION).lua
	./luarocks make --tree="$(DESTDIR)$(rocks_tree)"

# ----------------------------------------
# Windows binary build
# ----------------------------------------

windows-binary: luarocks
	$(MAKE) -f binary/Makefile.windows windows-binary

windows-clean:
	$(MAKE) -f binary/Makefile.windows windows-clean

# ----------------------------------------
# Clean
# ----------------------------------------

clean: windows-clean
	rm -rf ./config.unix \
		./luarocks \
		./luarocks-admin \
		$(builddir)/ \
		$(buildbinarydir)/ \
		./.luarocks \
		./lua_modules

.PHONY: all build install binary install-binary bootstrap clean windows-binary windows-clean
