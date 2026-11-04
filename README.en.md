# Passwall proxy packages feed

[简体中文](README.md) | **English**

This private source feed is used by the customized
[hnt510-great/openwrt](https://github.com/hnt510-great/openwrt) main tree.
The project targets x86_64/glibc, APK, native systemd and GNOME. Package
selection follows the main tree's saved configuration.

| Item | Value |
| --- | --- |
| Feed name | `passwall_packages` |
| Repository branch | `main` |
| Original upstream | [https://github.com/Openwrt-Passwall/openwrt-passwall-packages](https://github.com/Openwrt-Passwall/openwrt-passwall-packages) (`main`) |

This feed supplies proxy cores, DNS helpers and geodata used by Passwall.
Recipes include `xray-core`, `sing-box`, `hysteria`, `shadowsocks-rust`,
`chinadns-ng`, `smartdns` and `v2ray-geodata`. The management interface lives
in the separate `passwall_luci` feed.

## Main-tree integration

- The main image retains Passwall as its default proxy. `daed`, in
  `package/custom`, is an alternative that is disabled by default.
- `daed-geoip` and `daed-geosite` reuse this feed's `v2ray-geoip` and
  `v2ray-geosite` data.
- Native systemd retains UCI and init-script interfaces through the main
  tree's procd bridge. Consult its validation for proxy cgroups, monitoring
  and shutdown behavior.
- Proxied traffic and directly forwarded traffic follow different paths.
  Kernel full cone NAT results do not establish NAT behavior at a proxy exit.

## Get the source and build

Authenticate on the build machine with an account that can read every private
repository, then obtain the complete main tree:

```sh
gh auth login --hostname github.com --git-protocol https --web
gh auth setup-git
git clone https://github.com/hnt510-great/openwrt.git
cd openwrt
./scripts/feeds update -a
./scripts/feeds install -a
```

The main tree's `feeds.conf.default` already contains:

```text
src-git passwall_packages https://github.com/hnt510-great/openwrt-passwall-packages.git;main
```

To update only this feed in an existing main tree:

```sh
./scripts/feeds update passwall_packages
./scripts/feeds install -a -p passwall_packages
```

Select the desired package in the main configuration and run `make defconfig`.
Example individual package build:

```sh
make package/feeds/passwall_packages/xray-core/compile V=s
```

Run these commands from the main-tree root. Preserve an existing `.config`;
see [local build](https://github.com/hnt510-great/openwrt/blob/main/docs/local-build.md) for setup in a new checkout.
`install` creates build-recipe links rather than installing software on a
device. The full `update -a` also checks and can modify main-tree custom recipes.

## Maintenance and validation

`origin` points to this private repository. To merge original upstream changes
in a new clone, add `upstream` once:

```sh
git -C feeds/passwall_packages remote add upstream https://github.com/Openwrt-Passwall/openwrt-passwall-packages.git
git -C feeds/passwall_packages fetch upstream
git -C feeds/passwall_packages merge upstream/main
# Review, resolve conflicts and validate before pushing
git -C feeds/passwall_packages push origin main
```

Skip the add command if `upstream` already exists. `scripts/feeds update` pulls
the private branch; it does not merge original upstream automatically. Feed
integration and main-tree commits are separate. Preserve existing commits,
stashes and uncommitted work. Main-tree `patches/feeds/` files and feature
manifests are historical backups; do not reapply changes already present.

Changes to published APK contents require a new package version, preserving
immutable package identities. Record source checks, package builds, VM tests
and physical-device validation separately. This page summarizes existing
design and evidence without claiming new build or device results. See
[private GitHub hosting](https://github.com/hnt510-great/openwrt/blob/main/docs/github-source-hosting.md).

## Design and validation documents

- [Proxy alternatives and interlocks](https://github.com/hnt510-great/openwrt/blob/main/docs/daed.md)
- [systemd cgroups and Passwall validation](https://github.com/hnt510-great/openwrt/blob/main/docs/systemd-cgroups-validation.md)
- [DNS nftset fast path](https://github.com/hnt510-great/openwrt/blob/main/docs/dnsmasq-nftset.md)
- [Full cone NAT](https://github.com/hnt510-great/openwrt/blob/main/docs/fullcone-nat.md)

## License and upstream

Retain upstream copyrights and licenses. Each package's source follows its
recipe's `PKG_LICENSE` and source declarations.
