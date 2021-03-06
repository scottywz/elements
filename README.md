[Elements](https://www.youtube.com/watch?v=N0ziDSLJhq4)
=======================================================

[![](https://img.shields.io/badge/license-X11-green.svg?style=flat-square)
](LICENSE.txt)
[![](https://img.shields.io/badge/status-pre--alpha-red.svg?style=flat-square)
]()

A tool to generate single-file, runc-based AppImage containers.

Copyright (c) 2019 S. Zeid.  Some rights reserved under the X11 License.

<https://elements.s.zeid.me/>

**WARNING:**  Elements is not ready for production.  It is an experiment
in the early stages of development.  Although I make a best effort to not
commit broken code, there is no automated testing, there may be breaking
changes in future commits, there may be security or data loss bugs, and
there is no guarantee of long-term maintenance.  As the license states,
use at your own risk.  :)

*                        *                        *                        *

Elements is a tool to generate runc-based containers which are self-contained
in a single AppImage file.  The definition file can be used to specify if
and how parameters such as bind mounts and environment variables are passed
as arguments to the container's AppImage.


Current limitations
-------------------

Elements is currently a prototype written in a combination of Python 3 and
POSIX shell.  I plan to rewrite it in a compiled language other than Go,
but in the meantime there are some drawbacks to the current implementation:

* Currently, Elements uses an extended version of [Singularity Definition
  Files][sdf] to define containers and has a container build-time dependency
  on Singularity.  (Singularity is not used to run Elements containers, and
  unlike Singularity, **filesystems, PID and IPC namespaces, and the
  environment ARE contained by default**.)  
    
  This means that root is required to build the container image, layer
  caching is non-existent, and the definition format is completely different
  from Dockerfiles.  I plan to switch to Buildah and an extended (but
  backwards-compatible) Dockerfile syntax, which will resolve all these issues.

* At build time, Elements creates a POSIX-compliant (portable) shell script
  which is shipped in the AppImage.  This script parses the user-supplied
  arguments and constructs, runs, and tears down the underlying runc container
  at a temporary path.  This script relies on a minimal (~3 MB compressed)
  Alpine Linux rootfs, which is generated at container build time, is
  included in the AppImage, and contains jq for the purpose of configuring
  runc.  Shell was chosen because I'm currently not familiar enough with
  modern compiled languages, I wanted an MVP up and running as quickly as
  possible, and I did not want to ship an interpreter in the container image.
  Dollar sign variable syntax also comes for free with shell (although the
  builder sanitizes variables in the definition file when generating the
  script).
  
  The fact that this is a shell script, the script itself is inefficient, and
  it has to start up and tear down a container just for jq multiple times
  obviously results in slower startup times and increased attack surface.
  I plan to switch to a compiled runtime binary that includes its own
  (third-party) JSON library and also uses an intermediate config file to
  define the command-line arguments and other parts that are currently
  generated by the Python builder.  I may also decide to include extended
  support for POSIX parameter expansion (e.g. `${thing:-default}` to use
  a default value if `$thing` is unset or empty), which would use an
  in-house parser instead of relying on shell.


There is no timeline for the rewrite, and there is also no guarantee that it
will even happen.  If and when the rewrite is committed, it will _not_ be
backwards-compatible with current definition files.


[sdf]: https://www.sylabs.io/guides/3.0/user-guide/definition_files.html


## [Documentation](https://elements.s.zeid.me/)
