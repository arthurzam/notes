# Bumping Python Packages

## PEP517 mode

```bash
DISTUTILS_USE_PEP517=setuptools
```

## Enable network tests

```bash
RESTRICT="test"
PROPERTIES="test_network"
```

## Enable testing

```bash
distutils_enable_tests {nose,pytest,setup.py,unittest}
epytest
  EPYTEST_DESELECT=()
  EPYTEST_IGNORE=()
eunittest
```

For django testing it looks something like:

```bash
local -x DJANGO_SETTINGS_MODULE=
local -x PYTHONPATH="${S}"
django-admin test -v 2 || die
```

When we have issues with build native extensions and tests, use:

```bash
cd "${BUILD_DIR}/install$(python_get_sitedir)" || die
```

## `metadata.xml`

```xml
<pkgmetadata>
  <stabilize-allarches/>
  <upstream>
    <remote-id type="pypi"></remote-id>
    <remote-id type="github"></remote-id>
    <remote-id type="gitlab"></remote-id>
    <remote-id type="launchpad"></remote-id>
    <doc></doc>
    <bugs-to></bugs-to>
    <changelog></changelog>
  </upstream>
</pkgmetadata>
```

## feedlist

I like to use newsboat, as it has simple UI, while being very fast.

```bash
#!/bin/bash
set -e
set -o xtrace

tar czf ~/.newsboat/backup.tar.gz -C ~/.newsboat/ cache.db config urls arthurzam.opml

rm -v ~/.newsboat/urls
touch ~/.newsboat/urls

git -C ~/src/python/ pull
newsboat -i ~/src/python/release-feeds.opml
newsboat -i ~/.newsboat/arthurzam.opml
```

## Stable request workflow

```bash
stablereq-eshowkw 'dev-python/*'
stablereq-find-pkg-bugs 'dev-python/*' && stablereq-make-list 'dev-python/*'
```
