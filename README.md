# graphviz

Container images with [Graphviz](https://graphviz.org/), which lays out graphs written in the DOT language and draws them as SVG, PNG, PDF and many other formats. Graphviz is compiled from the release tarball on Ubuntu and Alpine, for `linux/amd64` and `linux/arm64`, with the cairo, Pango, gd, librsvg and WebP plugins and GTS for `sfdp`. The images are rebuilt when Graphviz publishes a release and when the base image changes.

This is an unofficial build, not affiliated with or endorsed by the Graphviz project. Report problems with the image in this repository and Graphviz bugs [upstream](https://gitlab.com/graphviz/graphviz/-/issues).

## Quick start

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/graphviz \
  -Tsvg -o graph.svg graph.gv
```

Graphviz reads the graph from standard input when no file is given, so a PNG can be made without a mount:

```sh
docker run --rm -i ghcr.io/randomcontainers/graphviz -Tpng -Gdpi=150 < graph.gv > graph.png
```

The entrypoint runs `dot` under `tini` in `/work`, so file names are relative to the directory you mount. Without arguments the image prints the Graphviz version. `-K` picks another layout engine, and `--entrypoint` runs one of the other tools:

```sh
# Lay out a large undirected graph with sfdp
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/graphviz \
  -Ksfdp -Tsvg -o network.svg network.gv

# Write graph.gv.pdf, named after the input
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/graphviz \
  -Tpdf -O graph.gv

# Remove transitive edges with tred
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint tred \
  ghcr.io/randomcontainers/graphviz graph.gv

# Color every node named "db" with a gvpr script
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint gvpr \
  ghcr.io/randomcontainers/graphviz -c 'N[name == "db"]{ color = "red" }' graph.gv
```

The [Graphviz documentation](https://graphviz.org/documentation/) covers the [command-line options](https://graphviz.org/doc/info/command.html), the attributes and the output formats.

## What is in the image

- `dot` and the other layout engines, `neato`, `fdp`, `sfdp`, `twopi`, `circo`, `osage` and `patchwork`, in `/usr/local/bin`. The engines are links to `dot`, which picks its layout from the name it is called by.
- The tools that come with Graphviz, among them `gvpr`, `gc`, `tred`, `acyclic`, `ccomps`, `sccmap`, `unflatten`, `gvpack`, `gvmap`, `edgepaint` and the GML, GraphML and GXL converters. The `gvpr` library scripts are in `/usr/local/share/graphviz/gvpr`, where `gvpr -f` finds them.
- The Graphviz libraries and these plugins, loaded at run time from `/usr/local/lib/graphviz`:
  - core: SVG, PostScript, EPS, JSON, DOT, xdot, plain text, image maps, FIG, PIC, POV-Ray and Tk.
  - cairo with Pango: PNG and PDF, and SVG, PostScript and EPS as `svg:cairo`, `ps:cairo` and `eps:cairo`. Pango measures the text for every output format.
  - gd: GIF, JPEG, WBMP, VRML and `png:gd`.
  - WebP, and `kitty` and `vt` for graphics in the terminal.
  - librsvg, for SVG files in the `image` attribute. Graphviz recognizes an SVG file only when it starts with an XML declaration such as `<?xml version="1.0"?>`. PNG, JPEG and GIF images are read through cairo and gd.
- GTS, which `sfdp` and the `prism` overlap removal of `neato` and `fdp` use for triangulation. The Ubuntu images use Ubuntu's `libgts` package. Alpine has no GTS package, so the Alpine images have GTS 0.7.6, compiled from its release tarball, in `/usr/local/lib`.
- expat, for HTML-like labels, and the distro's cairo, Pango, gd, librsvg, libwebp, FreeType, fontconfig and zlib.

Given an unknown format such as `-T?`, `dot` prints every format this build supports.

Not included: the X11, GTK and Qt viewers and editors (the `xlib` plugin, `gvedit`, `smyrna`), `mingle`, the language bindings (Python, Perl, Lua, Tcl and the other SWIG bindings), PDF and PostScript node images (Poppler and Ghostscript loaders), the LASi and gdk-pixbuf plugins, `dot_builtins`, `dot_sandbox`, which needs Python and bubblewrap, and the manual pages, documentation, sample graphs and C headers. The configure flags and the configure summary are in `/usr/local/share/randomcontainers/graphviz/buildinfo`.

### Fonts

Graphviz finds fonts through fontconfig, and the image has the DejaVu fonts. The default font, Times-Roman, maps to DejaVu Serif, Helvetica and `sans-serif` to DejaVu Sans, and Courier and `monospace` to DejaVu Sans Mono. `dot -v` prints which font file each `fontname` resolved to. For other fonts, install them in an image built on `slim`, as shown under [Extending the slim image](#extending-the-slim-image).

## Default or slim

Graphviz's default image adds no other tools, so `latest` and `slim` are the same image, with the contents listed above. Use `latest` to run it and the `slim` tags as a base for your own image. The default image of [Doxygen](https://github.com/randomcontainers/doxygen) is built on this Graphviz image, for the graphs Doxygen draws with `dot`.

## Tags

`<version>` is a Graphviz release such as `16.1.0`. `<minor>` and `<major>` are its shorter forms, `16.1` and `16`, and follow the newest release in that series. Each row lists the default tag and its `slim` twin, which point to the same image.

| Tags | Base |
|---|---|
| `latest`, `slim` | Ubuntu |
| `<version>`, `<version>-slim` | Ubuntu |
| `<minor>`, `<minor>-slim`, `<major>`, `<major>-slim` | Ubuntu |
| `ubuntu`, `slim-ubuntu` | Ubuntu |
| `<version>-ubuntu`, `<version>-slim-ubuntu` | Ubuntu |
| `<minor>-ubuntu`, `<minor>-slim-ubuntu`, `<major>-ubuntu`, `<major>-slim-ubuntu` | Ubuntu |
| `<version>-ubuntu26.04`, `<version>-slim-ubuntu26.04` | Ubuntu 26.04 |
| `alpine`, `slim-alpine` | Alpine |
| `<version>-alpine`, `<version>-slim-alpine` | Alpine |
| `<minor>-alpine`, `<minor>-slim-alpine`, `<major>-alpine`, `<major>-slim-alpine` | Alpine |
| `<version>-alpine3.24`, `<version>-slim-alpine3.24` | Alpine 3.24 |

The images are currently built on Ubuntu 26.04 and Alpine 3.24. Tags without a distro version move to the next distro release when the project does; tags ending in `ubuntu26.04` or `alpine3.24` stay on that release and are no longer rebuilt once the project moves to the next one. Every tag of the current Graphviz version, including the exact version, is rebuilt in place (see [Updates](#updates)), so pin a digest when you need the same bytes every time.

Graphviz releases a new major version every few months, and a major release can remove command-line options, attributes or output formats. Layouts can also change between releases. Pin a `<major>` or `<version>` tag when your builds compare or commit rendered output.

## Platforms

`linux/amd64` and `linux/arm64`, for both Ubuntu and Alpine. Both are compiled natively on GitHub-hosted runners, without emulation.

## Files and permissions

The working directory is `/work`. The image runs as UID 1000, and any other UID works too: `HOME` is then `/`, and caches go to `/cache`, which anyone can write to. How to get output files owned by you depends on how you run containers:

| Runtime | Flag |
|---|---|
| Docker on Linux (rootful), GitHub Actions | `--user "$(id -u):$(id -g)"` |
| Rootless Podman | `--userns=keep-id` |
| Rootless Docker | `--user 0:0` (root in the container is your user on the host) |
| Docker Desktop on macOS or Windows | none, file ownership is mapped for you |

With `--read-only`, add `--tmpfs /cache`, so that fontconfig can write its cache if it needs to.

## Untrusted files

A DOT file can make Graphviz read other files: node images in the `image` attribute, `shapefile`, and the directories in `imagepath`. Raster, PDF and PostScript output can embed those files, so a graph from someone else can pull files that the container can read into its output. `gvpr` scripts can run shell commands with `system()`; run only scripts you trust. Large or crafted graphs can also keep a layout busy for a long time and use a lot of memory. For graphs from unknown sources, take away what the container does not need:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  --network none --read-only --tmpfs /cache --cap-drop ALL --security-opt no-new-privileges \
  --memory 1g --cpus 1 --pids-limit 64 \
  ghcr.io/randomcontainers/graphviz -Tsvg -o untrusted.svg untrusted.gv
```

Mount only the directory the job needs, and stop jobs that run too long, for example with `timeout 60 docker run ...`. The images pick up a new Graphviz release about a day after it is published.

## Extending the slim image

Use a `slim` tag as the base for your own image. `slim`, `slim-ubuntu` and `slim-alpine` move to each new Graphviz release and are rebuilt when the base image changes. Switch to root to install more, then back:

```dockerfile
FROM ghcr.io/randomcontainers/graphviz:slim-ubuntu@sha256:...
USER root
RUN apt-get update \
 && apt-get install -y --no-install-recommends fonts-noto-core fonts-noto-cjk \
 && rm -rf /var/lib/apt/lists/*
USER 1000:1000
```

Graphviz finds fonts installed this way through fontconfig. On Alpine, start from `slim-alpine` and use `apk add --no-cache font-noto font-noto-cjk`. The entrypoint is `["tini", "--", "dot"]`; set your own `ENTRYPOINT` if your image runs something else. To pick up new Graphviz releases and base image fixes, let Dependabot or Renovate update the digest in your `FROM` line.

Everything the image adds is under `/usr/local`. `/usr/local/share/randomcontainers/graphviz/` holds the version, the source URLs, the build options, the license files and `runtime-deps`, the list of distro packages Graphviz needs at run time. The plugin list that `dot` reads, `/usr/local/lib/graphviz/config8`, is written at build time with `dot -c`. Run `dot -c` as root again if you add a Graphviz plugin.

## Verifying

Each image has a build provenance attestation from this repository's GitHub Actions run, signed by the shared build workflow in `randomcontainers/ci`:

```sh
gh attestation verify oci://ghcr.io/randomcontainers/graphviz:latest \
  --repo randomcontainers/graphviz --signer-repo randomcontainers/ci
```

Each platform image also carries an SPDX SBOM that lists every distro package with its version:

```sh
docker buildx imagetools inspect ghcr.io/randomcontainers/graphviz:latest --format '{{ json .SBOM }}'
```

Before compiling, the build checks the Graphviz tarball, and on Alpine the GTS tarball, against the SHA-256 recorded in `package.yml`.

## Updates

The project checks the [releases of graphviz/graphviz](https://gitlab.com/graphviz/graphviz/-/releases) on GitLab every 15 minutes. A release is picked up once it is 24 hours old. Its tarball is checked against the `graphviz-<version>.tar.xz.sha256` file that the release publishes next to it. Graphviz publishes no signatures, and the checksum file comes from the same GitLab package registry as the tarball. The new version and the tarball's SHA-256 are then committed to `package.yml`, and the images are rebuilt. Only the newest release is built; tags of older versions stay as they were last built.

The images of the current version are also rebuilt when the Ubuntu or Alpine base image changes and at least every 7 days, so distro security fixes reach the current tags.

GTS on Alpine is pinned to a version in `package.yml` and does not follow its upstream releases automatically. GTS 0.7.6 dates from 2006 and is its latest release.

## Building

```sh
docker build -f Dockerfile.ubuntu --target slim \
  --build-arg VERSION=<version> \
  --build-arg SOURCE_SHA256=<sha256 from package.yml> \
  -t graphviz:local .
```

For the Alpine image, use `Dockerfile.alpine` and also pass `--build-arg GTS_VERSION=<version>` and `--build-arg GTS_SHA256=<sha256>`, the `version` and `sha256` of the `gts` entry under `extra-artifacts` in `package.yml`. `--build-arg JOBS=<n>` limits the number of parallel compile jobs.

## Licenses

Graphviz is licensed under the Eclipse Public License 2.0 (EPL-2.0), with no secondary license. Parts of it have their own licenses:

- BSD-3-Clause: the ellipse code by Luc Maisonobe and the Mersenne Twister code in randomkit (`COPYING.ellipse`, `COPYING.randomkit`).
- CPL-1.0: the VPSC solver by Tim Dwyer, used by `neato -Gmode=ipsep` and the `vpsc` overlap removal, and the `gv2gml` converter (`COPYING.vpsc`).
- MIT: randomkit by Jean-Sebastien Roy (`COPYING.randomkit`).
- An Apache-style license from ColorBrewer, version 1.1: the ColorBrewer color schemes (`COPYING.brewer_colors`). This product includes color specifications and designs developed by Cynthia Brewer (http://colorbrewer.org/).
- A permissive notice from Emin Martinian, which has no SPDX identifier: the red-black tree code (`LICENSE.rbtree`).

Everything in the list is compiled into the Graphviz programs, libraries and plugins. The files named above are in `/usr/local/share/randomcontainers/graphviz/licenses/`, with Graphviz's `COPYING`. On Alpine the directory also holds `COPYING.gts`: GTS is licensed under the GNU Lesser General Public License, version 2 or later (LGPL-2.0-or-later). The image's license label is `EPL-2.0 AND BSD-3-Clause AND CPL-1.0 AND LGPL-2.0-or-later AND MIT AND LicenseRef-ColorBrewer AND LicenseRef-rbtree`, where the two `LicenseRef-` entries stand for the ColorBrewer license and the red-black tree notice. The Ubuntu and Alpine packages in the image keep their own licenses, for example LGPL for cairo, Pango, librsvg and, on Ubuntu, GTS.

The corresponding source for each image:

- Graphviz: every version has a GitHub release in this repository, named `v<version>`, with the exact `graphviz-<version>.tar.xz` that was compiled and the `gts-<version>.tar.gz` compiled on Alpine. The build applies no patches. `/usr/local/share/randomcontainers/graphviz/source` lists the release and the upstream URLs.
- Build scripts: this repository at the commit in the image's `org.opencontainers.image.revision` label. The Dockerfiles hold every configure flag.
- Ubuntu packages: the source packages on [Launchpad](https://launchpad.net/ubuntu) for the versions listed in the SBOM. `apt-get source <package>=<version>` fetches a version that is still in the Ubuntu archive.
- Alpine packages: Alpine has no source packages. For the versions listed in the SBOM, the source is the APKBUILD and patches in [aports](https://gitlab.alpinelinux.org/alpine/aports/-/tree/3.24-stable), branch `3.24-stable`, and the archives on [distfiles.alpinelinux.org](https://distfiles.alpinelinux.org/distfiles/v3.24/).

Combined images that include Graphviz are built on, or copy Graphviz from, a slim Graphviz image from this repository. Their `com.randomcontainers.members` label records the Graphviz version and the digest of that image, whose own `org.opencontainers.image.revision` label names the commit here.

The files in this repository are available under the MIT license, see [LICENSE](LICENSE).

## Requesting a tool

To suggest another tool, use the [Request a tool](https://github.com/randomcontainers/.github/issues/new?template=tool-request.yml) form.
