# Tales from using crossdev

## openjdk for sparc, built on ppc64le

I wanted to bootstrap an openjdk:11 tarball for sparc64, and was using crossdev
for it. I decided to use `bogsucker.ppc64` devbox, because it was quite under
utilized, and also - why not?

Well, this will be a long story, with hack and what not, so prepare yourself.

We start with basic setup of crossdev:

```bash
# create crossdev repo
mkdir -p /tmp/crossdev/metadata
cat > /tmp/crossdev/metadata/layout.conf <<- EOF
    masters = gentoo
    repo-name = crossdev
    use-manifests = true
    thin-manifests = true
EOF

# inform portage about our repos
mkdir -p /etc/portage/repos.conf
cat > /etc/portage/repos.conf/gentoo.conf <<- EOF
    [gentoo]
    location = /var/db/repos/gentoo
    auto-sync = no
    sync-type = git
EOF
cat > /etc/portage/repos.conf/crossdev.conf <<- EOF
    [crossdev]
    location = /tmp/crossdev
    auto-sync = no
EOF

# create crossdev
emerge sys-devel/crossdev
crossdev -S -t sparc64-unknown-linux-gnu
```

One big surprise you can wait for, is the fact that we already provide 4
packages using the crossdev installation, so I recommend to put them as
`package.provided`. In my case it was:

```
### /usr/sparc64-unknown-linux-gnu/etc/portage/profile/package.provided
sys-kernel/linux-headers-5.15-r3
sys-devel/binutils-2.38-r2
sys-devel/gcc-11.3.0
sys-libs/glibc-2.36-r5
```

Then do some modifications to `make.conf`, so it works better with sane
defaults:

```bash
MAKEOPTS="-j16"
PYTHON_TARGETS="python3_10"
PYTHON_SINGLE_TARGET="python3_10"
```

I also needed to disable globally `-pam` and `-introspection` USE flags.

For faster compilation times, I recommend to mount a `tmpfs` over tmpdir, and
you also need to mount `dev` for `app-text/qpdf` compilation to pass.

```bash
mount -t tmpfs tmpfs /usr/sparc64-unknown-linux-gnu/tmp/portage -o size=20G
mount --rbind /dev /usr/sparc64-unknown-linux-gnu/dev
```

Now, I'm at the stage that I can hope to cross build openjdk, so let's start
with `emerge dev-java/openjdk-bin:11` for the host - this one will be used as
builder for cross openjdk. Add needed USE flags for dependencies as needed.

After some experimentation with USE flags, in the cross environment I needed to
set those settings:

```
### /usr/sparc64-unknown-linux-gnu/etc/portage/package.use
app-text/ghostscript-gpl cups
app-crypt/gnupg ssl
sys-libs/pam -filecaps
dev-libs/libpcre2 -jit
dev-libs/libxml2 -python
```

Now we need to get to the preparations of bootstrapping

**To be continued**
