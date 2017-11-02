# urltest_webdav

Want to stress-test your WebDAV server?  This project is a C program that uses libcurl to perform a sequence of random-order WebDAV-style uploads of a directory/file to a remote URL.  The fine-grain timing features present in libcurl are used to generate timing statistics for each directory/file present.  The data points embody the time measured by libcurl to:
- perform DNS lookup of the target host
- open a TCP connection to the target host
- establish optional TLS/SSL encryption on the connection
- send the HTTP request
- begin reading the HTTP response
as well as
- total time for HTTP request and response
- bytes transferred for the HTTP response
Statistics for the requests are aggregated by:
- all HTTP status codes
- 200-level HTTP status codes
- 300-level HTTP status codes
- 400-level HTTP status codes
- 500-level HTTP status codes

The program can process one or more files or directories, mirroring them to a single URL.  For files, the file is:
1. Uploaded (`PUT`) to the base URL
2. Checked for properties (`PROPFIND`)
3. Downloaded (`GET`) from the remove URL (all received data discarded)
4. Removed (`DELETE`) from the remote URL
Directories are scanned to produce an in-memory representation that is processed in a semi-random fashion (versus being processed in a fully depth- or breadth-first order).  Files encountered are processed (eventually) in the same sequence as above; for directories the (eventual) sequence is:
1. Create the directory (`MKCOL`) at the remote URL
2. Upload all child entities
3. Check for all properties (`PROPFIND`)
4. Download all child entities
5. Download (`GET`) from the remove URL (received file listing is discarded)
6. Remove all child entities
7. Remove (`DELETE`) at the remote URL
This procedure is performed once by default, but can be repeated any number of times.  For example:

~~~~
$ ./urltest_webdav -u mud -p 'here is my passw0rd!' -U https://webdav.www.server.org/upload_dir -vnlg2 Makefile

Mirroring content to 'https://webdav.www.server.org/upload_dir'

Commencing 1 iteration...
201 📄 [0000 ↑] 8138     Makefile
207 📄 [0000 ℹ] 8138     Makefile
200 📄 [0000 ↓] 8138     Makefile
204 📄 [0000 ✖︎] 8138     Makefile
Generation 1 completed
201 📄 [0001 ↑] 8138     Makefile
207 📄 [0001 ℹ] 8138     Makefile
200 📄 [0001 ↓] 8138     Makefile
204 📄 [0001 ✖︎] 8138     Makefile
Generation 2 completed
~~~~
An example using this git repository directory itself:
~~~~
$ urltest_webdav -u mud -p 'here is my passw0rd!' -U https://webdav.www.server.org/upload_dir/urltest_webdav -vnl urltest_webdav

Mirroring content to 'https://webdav.www.udel.edu/www1_farm_webdav/urltest_webdav'

Commencing 1 iteration...
201 📁 [0000 ↑] 510      /tmp/urltest_webdav
201 📄 [0000 ↑] 1363     /tmp/urltest_webdav/http_stats.h
201 📄 [0000 ↑] 17315    /tmp/urltest_webdav/fs_entity.c
201 📄 [0000 ↑] 2324     /tmp/urltest_webdav/util_fns.c
207 📄 [0000 ℹ] 2324     /tmp/urltest_webdav/util_fns.c
207 📄 [0000 ℹ] 1363     /tmp/urltest_webdav/http_stats.h
201 📄 [0000 ↑] 1266     /tmp/urltest_webdav/http_ops.h
207 📄 [0000 ℹ] 17315    /tmp/urltest_webdav/fs_entity.c
201 📄 [0000 ↑] 370      /tmp/urltest_webdav/util_fns.h
207 📄 [0000 ℹ] 370      /tmp/urltest_webdav/util_fns.h
207 📄 [0000 ℹ] 1266     /tmp/urltest_webdav/http_ops.h
201 📄 [0000 ↑] 2856     /tmp/urltest_webdav/fs_entity.h
200 📄 [0000 ↓] 1363     /tmp/urltest_webdav/http_stats.h
200 📄 [0000 ↓] 2324     /tmp/urltest_webdav/util_fns.c
201 📄 [0000 ↑] 3594     /tmp/urltest_webdav/README.md
204 📄 [0000 ✖︎] 1363     /tmp/urltest_webdav/http_stats.h
201 📄 [0000 ↑] 1734     /tmp/urltest_webdav/CMakeLists.txt
207 📄 [0000 ℹ] 3594     /tmp/urltest_webdav/README.md
204 📄 [0000 ✖︎] 2324     /tmp/urltest_webdav/util_fns.c
201 📄 [0000 ↑] 11352    /tmp/urltest_webdav/http_ops.c
200 📄 [0000 ↓] 370      /tmp/urltest_webdav/util_fns.h
200 📄 [0000 ↓] 3594     /tmp/urltest_webdav/README.md
207 📄 [0000 ℹ] 11352    /tmp/urltest_webdav/http_ops.c
207 📄 [0000 ℹ] 2856     /tmp/urltest_webdav/fs_entity.h
204 📄 [0000 ✖︎] 370      /tmp/urltest_webdav/util_fns.h
200 📄 [0000 ↓] 11352    /tmp/urltest_webdav/http_ops.c
207 📄 [0000 ℹ] 1734     /tmp/urltest_webdav/CMakeLists.txt
201 📄 [0000 ↑] 7938     /tmp/urltest_webdav/http_stats.c
200 📄 [0000 ↓] 17315    /tmp/urltest_webdav/fs_entity.c
204 📄 [0000 ✖︎] 11352    /tmp/urltest_webdav/http_ops.c
204 📄 [0000 ✖︎] 3594     /tmp/urltest_webdav/README.md
201 📄 [0000 ↑] 12046    /tmp/urltest_webdav/urltest_webdav.c
204 📄 [0000 ✖︎] 17315    /tmp/urltest_webdav/fs_entity.c
200 📄 [0000 ↓] 1266     /tmp/urltest_webdav/http_ops.h
200 📄 [0000 ↓] 2856     /tmp/urltest_webdav/fs_entity.h
201 📁 [0000 ↑] 442      /tmp/urltest_webdav/.git
207 📄 [0000 ℹ] 12046    /tmp/urltest_webdav/urltest_webdav.c
200 📄 [0000 ↓] 1734     /tmp/urltest_webdav/CMakeLists.txt
204 📄 [0000 ✖︎] 1266     /tmp/urltest_webdav/http_ops.h
200 📄 [0000 ↓] 12046    /tmp/urltest_webdav/urltest_webdav.c
204 📄 [0000 ✖︎] 2856     /tmp/urltest_webdav/fs_entity.h
207 📄 [0000 ℹ] 7938     /tmp/urltest_webdav/http_stats.c
204 📄 [0000 ✖︎] 12046    /tmp/urltest_webdav/urltest_webdav.c
200 📄 [0000 ↓] 7938     /tmp/urltest_webdav/http_stats.c
201 📄 [0000 ↑] 107      /tmp/urltest_webdav/.git/packed-refs
204 📄 [0000 ✖︎] 7938     /tmp/urltest_webdav/http_stats.c
204 📄 [0000 ✖︎] 1734     /tmp/urltest_webdav/CMakeLists.txt
201 📁 [0000 ↑] 170      /tmp/urltest_webdav/.git/refs
201 📁 [0000 ↑] 136      /tmp/urltest_webdav/.git/logs
207 📄 [0000 ℹ] 107      /tmp/urltest_webdav/.git/packed-refs
201 📄 [0000 ↑] 73       /tmp/urltest_webdav/.git/description
201 📁 [0000 ↑] 238      /tmp/urltest_webdav/.git/objects
201 📁 [0000 ↑] 68       /tmp/urltest_webdav/.git/branches
207 📄 [0000 ℹ] 73       /tmp/urltest_webdav/.git/description
207 📁 [0000 ℹ] 68       /tmp/urltest_webdav/.git/branches
201 📁 [0000 ↑] 68       /tmp/urltest_webdav/.git/refs/tags
201 📄 [0000 ↑] 23       /tmp/urltest_webdav/.git/HEAD
201 📄 [0000 ↑] 137      /tmp/urltest_webdav/.git/index
201 📁 [0000 ↑] 102      /tmp/urltest_webdav/.git/refs/remotes
200 📁 [0000 ↓] 68       /tmp/urltest_webdav/.git/branches
200 📄 [0000 ↓] 107      /tmp/urltest_webdav/.git/packed-refs
204 📁 [0000 ✖︎] 68       /tmp/urltest_webdav/.git/branches
201 📁 [0000 ↑] 102      /tmp/urltest_webdav/.git/refs/remotes/origin
207 📁 [0000 ℹ] 68       /tmp/urltest_webdav/.git/refs/tags
201 📁 [0000 ↑] 408      /tmp/urltest_webdav/.git/hooks
201 📁 [0000 ↑] 102      /tmp/urltest_webdav/.git/refs/heads
201 📄 [0000 ↑] 189      /tmp/urltest_webdav/.git/hooks/post-update.sample
204 📄 [0000 ✖︎] 107      /tmp/urltest_webdav/.git/packed-refs
207 📄 [0000 ℹ] 23       /tmp/urltest_webdav/.git/HEAD
201 📁 [0000 ↑] 102      /tmp/urltest_webdav/.git/info
200 📁 [0000 ↓] 68       /tmp/urltest_webdav/.git/refs/tags
201 📁 [0000 ↑] 102      /tmp/urltest_webdav/.git/objects/cc
201 📄 [0000 ↑] 314      /tmp/urltest_webdav/.git/config
201 📄 [0000 ↑] 32       /tmp/urltest_webdav/.git/refs/remotes/origin/HEAD
201 📄 [0000 ↑] 240      /tmp/urltest_webdav/.git/info/exclude
200 📄 [0000 ↓] 73       /tmp/urltest_webdav/.git/description
207 📄 [0000 ℹ] 240      /tmp/urltest_webdav/.git/info/exclude
201 📁 [0000 ↑] 136      /tmp/urltest_webdav/.git/logs/refs
200 📄 [0000 ↓] 240      /tmp/urltest_webdav/.git/info/exclude
201 📁 [0000 ↑] 68       /tmp/urltest_webdav/.git/objects/pack
204 📄 [0000 ✖︎] 73       /tmp/urltest_webdav/.git/description
204 📄 [0000 ✖︎] 240      /tmp/urltest_webdav/.git/info/exclude
201 📁 [0000 ↑] 102      /tmp/urltest_webdav/.git/objects/27
200 📄 [0000 ↓] 23       /tmp/urltest_webdav/.git/HEAD
207 📁 [0000 ℹ] 102      /tmp/urltest_webdav/.git/info
201 📁 [0000 ↑] 68       /tmp/urltest_webdav/.git/objects/info
207 📁 [0000 ℹ] 68       /tmp/urltest_webdav/.git/objects/pack
207 📄 [0000 ℹ] 314      /tmp/urltest_webdav/.git/config
201 📄 [0000 ↑] 41       /tmp/urltest_webdav/.git/refs/heads/master
201 📄 [0000 ↑] 544      /tmp/urltest_webdav/.git/hooks/pre-receive.sample
201 📄 [0000 ↑] 3610     /tmp/urltest_webdav/.git/hooks/update.sample
200 📄 [0000 ↓] 314      /tmp/urltest_webdav/.git/config
207 📄 [0000 ℹ] 137      /tmp/urltest_webdav/.git/index
204 📄 [0000 ✖︎] 314      /tmp/urltest_webdav/.git/config
201 📄 [0000 ↑] 4951     /tmp/urltest_webdav/.git/hooks/pre-rebase.sample
201 📄 [0000 ↑] 129      /tmp/urltest_webdav/.git/objects/cc/99ec94550ef9e931aa89dee127019702267f44
200 📁 [0000 ↓] 68       /tmp/urltest_webdav/.git/objects/pack
207 📄 [0000 ℹ] 32       /tmp/urltest_webdav/.git/refs/remotes/origin/HEAD
200 📄 [0000 ↓] 32       /tmp/urltest_webdav/.git/refs/remotes/origin/HEAD
204 📄 [0000 ✖︎] 32       /tmp/urltest_webdav/.git/refs/remotes/origin/HEAD
204 📄 [0000 ✖︎] 23       /tmp/urltest_webdav/.git/HEAD
201 📄 [0000 ↑] 514      /tmp/urltest_webdav/.git/objects/27/02d1ae8d62e26e5fba4c5895d3e0e16c2b98c6
201 📁 [0000 ↑] 102      /tmp/urltest_webdav/.git/objects/b8
201 📄 [0000 ↑] 185      /tmp/urltest_webdav/.git/logs/HEAD
200 📄 [0000 ↓] 137      /tmp/urltest_webdav/.git/index
201 📁 [0000 ↑] 102      /tmp/urltest_webdav/.git/logs/refs/remotes
207 📄 [0000 ℹ] 129      /tmp/urltest_webdav/.git/objects/cc/99ec94550ef9e931aa89dee127019702267f44
200 📁 [0000 ↓] 102      /tmp/urltest_webdav/.git/info
204 📁 [0000 ✖︎] 102      /tmp/urltest_webdav/.git/info
207 📄 [0000 ℹ] 41       /tmp/urltest_webdav/.git/refs/heads/master
207 📁 [0000 ℹ] 68       /tmp/urltest_webdav/.git/objects/info
201 📁 [0000 ↑] 102      /tmp/urltest_webdav/.git/logs/refs/heads
200 📄 [0000 ↓] 129      /tmp/urltest_webdav/.git/objects/cc/99ec94550ef9e931aa89dee127019702267f44
201 📄 [0000 ↑] 54       /tmp/urltest_webdav/.git/objects/b8/1fe69cf715eb1065659aad4cecfd8f2fe284bf
204 📁 [0000 ✖︎] 68       /tmp/urltest_webdav/.git/objects/pack
200 📄 [0000 ↓] 41       /tmp/urltest_webdav/.git/refs/heads/master
207 📁 [0000 ℹ] 102      /tmp/urltest_webdav/.git/refs/remotes/origin
201 📁 [0000 ↑] 102      /tmp/urltest_webdav/.git/logs/refs/remotes/origin
207 📄 [0000 ℹ] 185      /tmp/urltest_webdav/.git/logs/HEAD
204 📁 [0000 ✖︎] 68       /tmp/urltest_webdav/.git/refs/tags
200 📁 [0000 ↓] 68       /tmp/urltest_webdav/.git/objects/info
204 📄 [0000 ✖︎] 137      /tmp/urltest_webdav/.git/index
207 📄 [0000 ℹ] 54       /tmp/urltest_webdav/.git/objects/b8/1fe69cf715eb1065659aad4cecfd8f2fe284bf
201 📄 [0000 ↑] 1239     /tmp/urltest_webdav/.git/hooks/prepare-commit-msg.sample
201 📄 [0000 ↑] 896      /tmp/urltest_webdav/.git/hooks/commit-msg.sample
201 📄 [0000 ↑] 1642     /tmp/urltest_webdav/.git/hooks/pre-commit.sample
201 📄 [0000 ↑] 185      /tmp/urltest_webdav/.git/logs/refs/remotes/origin/HEAD
207 📄 [0000 ℹ] 514      /tmp/urltest_webdav/.git/objects/27/02d1ae8d62e26e5fba4c5895d3e0e16c2b98c6
207 📄 [0000 ℹ] 3610     /tmp/urltest_webdav/.git/hooks/update.sample
200 📁 [0000 ↓] 102      /tmp/urltest_webdav/.git/refs/remotes/origin
207 📄 [0000 ℹ] 185      /tmp/urltest_webdav/.git/logs/refs/remotes/origin/HEAD
204 📄 [0000 ✖︎] 41       /tmp/urltest_webdav/.git/refs/heads/master
204 📁 [0000 ✖︎] 68       /tmp/urltest_webdav/.git/objects/info
204 📁 [0000 ✖︎] 102      /tmp/urltest_webdav/.git/refs/remotes/origin
200 📄 [0000 ↓] 3610     /tmp/urltest_webdav/.git/hooks/update.sample
207 📄 [0000 ℹ] 1239     /tmp/urltest_webdav/.git/hooks/prepare-commit-msg.sample
200 📄 [0000 ↓] 54       /tmp/urltest_webdav/.git/objects/b8/1fe69cf715eb1065659aad4cecfd8f2fe284bf
207 📄 [0000 ℹ] 896      /tmp/urltest_webdav/.git/hooks/commit-msg.sample
200 📄 [0000 ↓] 185      /tmp/urltest_webdav/.git/logs/refs/remotes/origin/HEAD
207 📁 [0000 ℹ] 102      /tmp/urltest_webdav/.git/refs/remotes
204 📄 [0000 ✖︎] 129      /tmp/urltest_webdav/.git/objects/cc/99ec94550ef9e931aa89dee127019702267f44
200 📄 [0000 ↓] 514      /tmp/urltest_webdav/.git/objects/27/02d1ae8d62e26e5fba4c5895d3e0e16c2b98c6
204 📄 [0000 ✖︎] 185      /tmp/urltest_webdav/.git/logs/refs/remotes/origin/HEAD
200 📄 [0000 ↓] 185      /tmp/urltest_webdav/.git/logs/HEAD
204 📄 [0000 ✖︎] 185      /tmp/urltest_webdav/.git/logs/HEAD
200 📁 [0000 ↓] 102      /tmp/urltest_webdav/.git/refs/remotes
207 📄 [0000 ℹ] 1642     /tmp/urltest_webdav/.git/hooks/pre-commit.sample
207 📁 [0000 ℹ] 102      /tmp/urltest_webdav/.git/refs/heads
207 📄 [0000 ℹ] 4951     /tmp/urltest_webdav/.git/hooks/pre-rebase.sample
207 📁 [0000 ℹ] 102      /tmp/urltest_webdav/.git/logs/refs/remotes/origin
201 📄 [0000 ↑] 1348     /tmp/urltest_webdav/.git/hooks/pre-push.sample
204 📄 [0000 ✖︎] 3610     /tmp/urltest_webdav/.git/hooks/update.sample
200 📁 [0000 ↓] 102      /tmp/urltest_webdav/.git/logs/refs/remotes/origin
200 📄 [0000 ↓] 1642     /tmp/urltest_webdav/.git/hooks/pre-commit.sample
204 📄 [0000 ✖︎] 54       /tmp/urltest_webdav/.git/objects/b8/1fe69cf715eb1065659aad4cecfd8f2fe284bf
204 📄 [0000 ✖︎] 1642     /tmp/urltest_webdav/.git/hooks/pre-commit.sample
201 📄 [0000 ↑] 424      /tmp/urltest_webdav/.git/hooks/pre-applypatch.sample
200 📄 [0000 ↓] 896      /tmp/urltest_webdav/.git/hooks/commit-msg.sample
204 📁 [0000 ✖︎] 102      /tmp/urltest_webdav/.git/logs/refs/remotes/origin
204 📁 [0000 ✖︎] 102      /tmp/urltest_webdav/.git/refs/remotes
204 📄 [0000 ✖︎] 514      /tmp/urltest_webdav/.git/objects/27/02d1ae8d62e26e5fba4c5895d3e0e16c2b98c6
200 📄 [0000 ↓] 4951     /tmp/urltest_webdav/.git/hooks/pre-rebase.sample
207 📁 [0000 ℹ] 102      /tmp/urltest_webdav/.git/objects/27
207 📁 [0000 ℹ] 102      /tmp/urltest_webdav/.git/logs/refs/remotes
201 📄 [0000 ↑] 185      /tmp/urltest_webdav/.git/logs/refs/heads/master
200 📁 [0000 ↓] 102      /tmp/urltest_webdav/.git/refs/heads
200 📁 [0000 ↓] 102      /tmp/urltest_webdav/.git/logs/refs/remotes
204 📁 [0000 ✖︎] 102      /tmp/urltest_webdav/.git/refs/heads
207 📁 [0000 ℹ] 170      /tmp/urltest_webdav/.git/refs
200 📁 [0000 ↓] 170      /tmp/urltest_webdav/.git/refs
204 📁 [0000 ✖︎] 102      /tmp/urltest_webdav/.git/logs/refs/remotes
200 📄 [0000 ↓] 1239     /tmp/urltest_webdav/.git/hooks/prepare-commit-msg.sample
207 📄 [0000 ℹ] 185      /tmp/urltest_webdav/.git/logs/refs/heads/master
207 📄 [0000 ℹ] 1348     /tmp/urltest_webdav/.git/hooks/pre-push.sample
204 📁 [0000 ✖︎] 170      /tmp/urltest_webdav/.git/refs
204 📄 [0000 ✖︎] 1239     /tmp/urltest_webdav/.git/hooks/prepare-commit-msg.sample
207 📁 [0000 ℹ] 102      /tmp/urltest_webdav/.git/objects/cc
200 📄 [0000 ↓] 185      /tmp/urltest_webdav/.git/logs/refs/heads/master
204 📄 [0000 ✖︎] 185      /tmp/urltest_webdav/.git/logs/refs/heads/master
207 📁 [0000 ℹ] 102      /tmp/urltest_webdav/.git/logs/refs/heads
204 📄 [0000 ✖︎] 4951     /tmp/urltest_webdav/.git/hooks/pre-rebase.sample
200 📁 [0000 ↓] 102      /tmp/urltest_webdav/.git/logs/refs/heads
204 📁 [0000 ✖︎] 102      /tmp/urltest_webdav/.git/logs/refs/heads
207 📁 [0000 ℹ] 136      /tmp/urltest_webdav/.git/logs/refs
200 📁 [0000 ↓] 136      /tmp/urltest_webdav/.git/logs/refs
200 📁 [0000 ↓] 102      /tmp/urltest_webdav/.git/objects/27
204 📁 [0000 ✖︎] 102      /tmp/urltest_webdav/.git/objects/27
204 📁 [0000 ✖︎] 136      /tmp/urltest_webdav/.git/logs/refs
207 📁 [0000 ℹ] 102      /tmp/urltest_webdav/.git/objects/b8
207 📁 [0000 ℹ] 136      /tmp/urltest_webdav/.git/logs
200 📁 [0000 ↓] 136      /tmp/urltest_webdav/.git/logs
204 📁 [0000 ✖︎] 136      /tmp/urltest_webdav/.git/logs
200 📁 [0000 ↓] 102      /tmp/urltest_webdav/.git/objects/b8
204 📁 [0000 ✖︎] 102      /tmp/urltest_webdav/.git/objects/b8
200 📁 [0000 ↓] 102      /tmp/urltest_webdav/.git/objects/cc
207 📄 [0000 ℹ] 424      /tmp/urltest_webdav/.git/hooks/pre-applypatch.sample
200 📄 [0000 ↓] 424      /tmp/urltest_webdav/.git/hooks/pre-applypatch.sample
204 📄 [0000 ✖︎] 896      /tmp/urltest_webdav/.git/hooks/commit-msg.sample
201 📄 [0000 ↑] 478      /tmp/urltest_webdav/.git/hooks/applypatch-msg.sample
204 📄 [0000 ✖︎] 424      /tmp/urltest_webdav/.git/hooks/pre-applypatch.sample
207 📄 [0000 ℹ] 478      /tmp/urltest_webdav/.git/hooks/applypatch-msg.sample
207 📄 [0000 ℹ] 189      /tmp/urltest_webdav/.git/hooks/post-update.sample
204 📁 [0000 ✖︎] 102      /tmp/urltest_webdav/.git/objects/cc
200 📄 [0000 ↓] 478      /tmp/urltest_webdav/.git/hooks/applypatch-msg.sample
207 📁 [0000 ℹ] 238      /tmp/urltest_webdav/.git/objects
200 📄 [0000 ↓] 189      /tmp/urltest_webdav/.git/hooks/post-update.sample
200 📄 [0000 ↓] 1348     /tmp/urltest_webdav/.git/hooks/pre-push.sample
200 📁 [0000 ↓] 238      /tmp/urltest_webdav/.git/objects
204 📁 [0000 ✖︎] 238      /tmp/urltest_webdav/.git/objects
204 📄 [0000 ✖︎] 1348     /tmp/urltest_webdav/.git/hooks/pre-push.sample
207 📄 [0000 ℹ] 544      /tmp/urltest_webdav/.git/hooks/pre-receive.sample
204 📄 [0000 ✖︎] 189      /tmp/urltest_webdav/.git/hooks/post-update.sample
200 📄 [0000 ↓] 544      /tmp/urltest_webdav/.git/hooks/pre-receive.sample
204 📄 [0000 ✖︎] 478      /tmp/urltest_webdav/.git/hooks/applypatch-msg.sample
204 📄 [0000 ✖︎] 544      /tmp/urltest_webdav/.git/hooks/pre-receive.sample
207 📁 [0000 ℹ] 408      /tmp/urltest_webdav/.git/hooks
200 📁 [0000 ↓] 408      /tmp/urltest_webdav/.git/hooks
204 📁 [0000 ✖︎] 408      /tmp/urltest_webdav/.git/hooks
207 📁 [0000 ℹ] 442      /tmp/urltest_webdav/.git
200 📁 [0000 ↓] 442      /tmp/urltest_webdav/.git
204 📁 [0000 ✖︎] 442      /tmp/urltest_webdav/.git
207 📁 [0000 ℹ] 510      /tmp/urltest_webdav
200 📁 [0000 ↓] 510      /tmp/urltest_webdav
204 📁 [0000 ✖︎] 510      /tmp/urltest_webdav
Generation 1 completed
~~~~
Were this same command to be repeated, the order of file/directory processing would look different.

Command line options are present to alter the verbosity of the program, provide static hostname-to-IP mappings, set HTTP basic authentication parameters, and trigger a dry-run testing (no actual HTTP transactions).

~~~~
$ ./urltest_webdav -h
version 1.0.0
built Nov  2 2017 16:59:21
usage:

  ./urltest_webdav {options} <file/directory> {<file/directory> ..}

 options:

  --help/-h                    show this information

  --long-listing/-l            list the discovered file hierarchy in an extended
                               format
  --short-listing/-s           list the discovered file hierarchy in a compact
                               format
  --no-listing/-n              do not list the discovered file hierarchy
  --ascii/-a                   restrict to ASCII characters

  --verbose/-v                 display additional information to stdout as the
                               program progresses
  --dry-run/-d                 do not perform any HTTP requests, just show an
                               activity trace
  --show-timings/-t            show HTTP timing statistics at the end of the run
  --generations/-g <#>         maximum number of generations to iterate

  --base-url/-U <URL>          the base URL to which the content should be mirrored
  --host-mapping/-m <hostmap>  provide a static DNS mapping for a hostname and TCP/IP
                               port

                                 <hostmap> = <hostname>:<port>:<ip address>

  --username/-u <string>       use HTTP basic authentication with the given string as
                               the username
  --password/-p <string>       use HTTP basic authentication with the given string as
                               the password

~~~~
