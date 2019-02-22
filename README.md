# Build pgAudit on Windows

This describes how you can build pgAudit on Windows.

## Introduction and disclaimer

The content herein is based on compiling pgAudit 1.3.0 against PostgreSQL 11.2.

It may work for you, and it may work on older and on newer releases. Then, it may not work.

The best way (in terms of build quality) to build pgAudit is to build it as part of the contrib tree in a full build
of PostgreSQL. However, it can be done without a full build.

## Build pgAudit as part of a full PostgreSQL build

Download [the source code for PostgreSQL 11.2](https://www.postgresql.org/ftp/source/v11.2/).

Unpack the full source tree somewhere on your build machine.

Download the [source for pgAudit 1.3.0](https://github.com/pgaudit/pgaudit/releases).

Unpack this directly below the contrib subtree.

See [Chapter 17](https://www.postgresql.org/docs/11/install-windows.html) in the documentation regarding this.
[Visual Studio 2017 Community Edition](https://visualstudio.microsoft.com/vs/community/) is a fine build environment
to do the job. Make sure that Workload Desktop development with C++ is installed and that Windows 8.1 SDK is installed. 
This SDK has needed header files included that later SDK's do not.

Install additional requirements as described in
[Chapter 17.1.1](https://www.postgresql.org/docs/11/install-windows-full.html#id-1.6.4.8.8)
of the documentation. Not all are strictly needed for a default build.
Start with installing Perl and try a default build with that. If that fails, then fix each failure and try again.

After a successful build, copy pgaudit.dll from Release\pgaudit and install using that.

## Install pgAudit from a full PostgreSQL build

This assumes a PostgreSQL binary installation on Windows using default file locations.

* Copy __pgaudit.dll__ to `C:\Program Files\PostgreSQL\11\lib`.
  
* Copy __pgaudit.control__ and __pgaudit--1.3.sql__ from the pgaudit distribution
to `C:\Program Files\PostgreSQL\11\share\extension`.

* Set `shared_preload_libraries = pgaudit` in postgresql.conf.

* Restart PostgreSQL.

* set `pgaudit.log = all` in postgresql.conf.

* Reload the configuration, e.g. by doing a `select pg_reload_conf();` as a superuser.

* Watch the log grow.

* Drink coffee.

## Build pgAudit against a binary installation of PostgreSQL

The full PostgreSQL build has a lot of automatic generation of build options included.
The build of pgaudit is no exception. The instructions below are based on what the full build process generates.

Assumptions below are default locations of PostgreSQL.

* Create a build-folder somewhere.

* Copy __pgaudit.c__ to this folder.

* Open a `x64 Native Tools Command Prompt for VS 2017`.

* Go to the folder with __pgaudit.c__.

* Create a __pgaudit.def__ file. Put the text below in the file.

```
EXPORTS
  Pg_magic_func
  _PG_init
  auditEventStack DATA
  auditLog DATA
  auditLogCatalog DATA
  auditLogClient DATA
  auditLogLevel DATA
  auditLogLevelString DATA
  auditLogParameter DATA
  auditLogRelation DATA
  auditLogStatementOnce DATA
  auditRole DATA
  pg_finfo_pgaudit_ddl_command_end
  pg_finfo_pgaudit_sql_drop
  pgaudit_ddl_command_end
  pgaudit_sql_drop
```

* Create a __compile_response_file.txt__ file. Put the text below in the file.

```
 /I"C:\Program Files\PostgreSQL\11\include\server"
 /I"C:\Program Files\PostgreSQL\11\include\server\port\win32"
 /I"C:\Program Files\PostgreSQL\11\include\server\port\win32_msvc"
 /I"C:\Program Files\PostgreSQL\11\include"
 /Zi
 /nologo
 /W3
 /WX-
 /diagnostics:classic
 /Ox
 /D WIN32
 /D _WINDOWS
 /D __WINDOWS__
 /D __WIN32__
 /D EXEC_BACKEND
 /D WIN32_STACK_RLIMIT=4194304
 /D _CRT_SECURE_NO_DEPRECATE
 /D _CRT_NONSTDC_NO_DEPRECATE
 /D _WINDLL
 /D _MBCS
 /GF
 /Gm-
 /EHsc
 /MD
 /GS
 /fp:precise
 /Zc:wchar_t
 /Zc:forScope
 /Zc:inline
 /Gd
 /TC
 /wd4018
 /wd4244
 /wd4273
 /wd4102
 /wd4090
 /wd4267
 /FC
 /errorReport:queue
```

* Create a __link_response_file.txt__ file. Put the text below in the file.

```
/ERRORREPORT:QUEUE
/INCREMENTAL:NO
/NOLOGO
"C:\Program Files\PostgreSQL\11\lib\postgres.lib"
/NODEFAULTLIB:libc
/DEF:".\pgaudit.def"
/MANIFEST
/MANIFESTUAC:"level='asInvoker' uiAccess='false'"
/manifest:embed
/DEBUG
/SUBSYSTEM:CONSOLE
/STACK:"4194304"
/TLBID:1
/DYNAMICBASE:NO
/NXCOMPAT
/IMPLIB:".\pgaudit.lib"
/MACHINE:X64
/ignore:4197
/DLL
```

* Build, using the compile and link commands below.

```
cl /c @.\compile_response_file.txt .\pgaudit.c
link.exe @.\link_response_file.txt /OUT:".\pgaudit.dll" .\pgaudit.obj
```

## Install pgAudit from a separate pgAudit build

Follow the same steps for installing the newly compiled __pgaudit.dll__ as described with the full PostgreSQL build.

## Basic troubleshooting

Begin with confirming that pgAudit is loaded and that logging is enabled. Logon using `psql`.

```
localhost postgres@postgres#show shared_preload_libraries;
 shared_preload_libraries
--------------------------
 pgaudit
(1 row)

localhost postgres@postgres#show pgaudit.log;
 pgaudit.log
-------------
 all
(1 row)

localhost postgres@postgres#show log_destination;
 log_destination
-----------------
 stderr
```

Inspect the logfile. There should be new entries.

## Pull build instructions from a full PostgreSQL build log

When you do a full PostgreSQL build, output from the build process scrolls across the screen. After completion
you can copy everything from the console window with `Ctrl-a`, `Ctrl-c` and put this into an editor. Search for 
__pgaudit__ and closely inspect what goes on there. Compiler switches can be found here, and the __.def__ file can 
be found and copied as well. Reuse the results for a separate build.
