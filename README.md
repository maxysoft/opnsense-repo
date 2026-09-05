# opnsense-repo

Custom OPNsense plugin repository. Packages are built from source on FreeBSD in
CI and published as a signed-by-TLS `pkg` repository on GitHub Pages, so plugins
show up in **System → Firmware → Plugins** like any official one.

## Install the repository

On the firewall, as root:

```
fetch -o /usr/local/etc/pkg/repos/mxy-opnsense-repo.conf https://maxysoft.github.io/opnsense-repo/mxy-opnsense-repo.conf
pkg update
```

Then install from the GUI (System → Firmware → Plugins), or from the shell:

```
pkg install os-devicemonitor
```

See what the repo offers, and what you already have from it:

```
pkg search -g -r maxysoft \*
pkg query -a '%R %n-%v' | grep maxysoft
```

Remove the repository (previously installed packages are kept):

```
rm /usr/local/etc/pkg/repos/mxy-opnsense-repo.conf
```

## Packages

| package | source | original author | OPNsense |
| --- | --- | --- | --- |
| `os-devicemonitor` | [maxysoft/opnsense-devicemonitor](https://github.com/maxysoft/opnsense-devicemonitor) | [Hacesoft](https://github.com/hacesoft) | 26.1, 26.7 |

## Credits

This repository only packages and distributes plugins; it does not author them.
Device Monitor was written by **[Hacesoft](https://github.com/hacesoft)**
([hacesoft.cz](https://hacesoft.cz)). See
[hacesoft/opnsense-devicemonitor](https://github.com/hacesoft/opnsense-devicemonitor)
for the upstream project. Please send thanks, and any report about the plugin's
own behaviour, upstream; open issues here only for packaging problems.

## How it is built

`.github/workflows/build.yml` on every push to `master`:

1. resolves `PLUGIN_VERSION` to the matching tag in the plugin repo and checks
   the plugin out at that tag, so a version always identifies one commit,
2. checks out the OPNsense plugin build framework (`opnsense/plugins`) and
   `opnsense/core` for the lint rules, both pinned to `OPNSENSE_PLUGINS_REF`,
3. assembles them into a throwaway ports tree under `tree/`,
4. in a FreeBSD VM: runs the official lint targets and `php -l`, compiles the
   `.po` translations, runs `make package`, installs and removes the result to
   prove it works, pulls previously released packages back in, then runs
   `pkg repo` over `site/repo/${ABI}/`,
5. publishes every built package as a GitHub Release,
6. merges the per-series results and publishes `site/` to GitHub Pages.

The built packages are **not stored in git**, they are published to the Pages
deployment and attached to Releases. `site/repo/` is gitignored because it is
generated on every run; git holds only the sources needed to reproduce it.

### Supported OPNsense series

One package is built per series, because each is based on a different FreeBSD
major and therefore a different pkg ABI. `${ABI}` in `mxy-opnsense-repo.conf` makes each
firewall pick its own automatically.

| OPNsense | FreeBSD | pkg ABI |
| --- | --- | --- |
| 26.7 | 15.0 | `FreeBSD:15:amd64` |
| 26.1 | 14.3 | `FreeBSD:14:amd64` |

When OPNsense starts a new series, add a row to the `build` job's matrix with
the FreeBSD release that series is built on. Getting that pairing wrong
produces a repository the firewall silently ignores.

### Adding a plugin

Create `<category>/<name>/{Makefile,pkg-descr}` following the existing one, add
a checkout step for its source, and extend the assemble/build steps. The plugin
`Makefile` needs only `PLUGIN_NAME`, `PLUGIN_VERSION` and `PLUGIN_COMMENT`;
everything else has a sane default in `Mk/plugins.mk`.

### Installing a specific version, or rolling back

The repository catalogue carries every version that has been released, not just
the newest, so the plugin manager and `pkg upgrade` see the latest while older
builds stay installable.

Every build is also attached to a [GitHub
Release](https://github.com/maxysoft/opnsense-repo/releases), which is the
permanent archive. To pin or roll back, install the asset directly:

```
pkg add https://github.com/maxysoft/opnsense-repo/releases/download/v2.3/os-devicemonitor-2.3-FreeBSD_15_amd64.pkg
```

Use the `FreeBSD_14_amd64` asset on OPNsense 26.1. Add `-f` to force a
downgrade over a newer installed version, and remember that `pkg upgrade` will
move it forward again unless you `pkg lock os-devicemonitor`.

### Releasing a new plugin version

Tag the plugin repo at the commit to release (`git tag -a v2.4 -m 'Device
Monitor 2.4' && git push origin v2.4`), then bump `PLUGIN_VERSION` in
`net-mgmt/devicemonitor/Makefile` to match and push here. The build fails with a
clear message if the tag does not exist, and fails if `PLUGIN_VERSION` and the
`version` field in the plugin's `defaults.json` disagree, so the package version
cannot silently drift from the version
the plugin reports at runtime.

## Signing

The repository is unsigned; trust rests on HTTPS, same as
[mimugmail/opn-repo](https://github.com/mimugmail/opn-repo). To sign it later:

1. `openssl genrsa -out repo.key 4096 && openssl rsa -in repo.key -pubout -out repo.pub`
2. store `repo.key` as an actions secret, write it to a file in the VM step,
3. `pkg repo "$REPO_DIR" /path/to/repo.key`,
4. publish `repo.pub` alongside `mxy-opnsense-repo.conf`,
5. add to `mxy-opnsense-repo.conf`: `signature_type: "pubkey"`, `pubkey: "/usr/local/etc/pkg/keys/maxysoft.pub"`,
6. document that users must fetch `repo.pub` to that path before `pkg update`.

## Licence

BSD 3-Clause. See [LICENSE](LICENSE). This covers the packaging in this
repository only; each packaged plugin keeps its own licence, granted by its
original author.
