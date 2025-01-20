---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/python/version/","noteIcon":"3"}
---

#version #release

基于setuptools 69.0版本`setuptools/_vendor/packaging/version.py`
```py
import re

_VERSION_PATTERN = r"""
    v?
    (?:
        (?:(?P<epoch>[0-9]+)!)?                           # epoch
        (?P<release>[0-9]+(?:\.[0-9]+)*)                  # release segment
        (?P<pre>                                          # pre-release
            [-_\.]?
            (?P<pre_l>(a|b|c|rc|alpha|beta|pre|preview))
            [-_\.]?
            (?P<pre_n>[0-9]+)?
        )?
        (?P<post>                                         # post release
            (?:-(?P<post_n1>[0-9]+))
            |
            (?:
                [-_\.]?
                (?P<post_l>post|rev|r)
                [-_\.]?
                (?P<post_n2>[0-9]+)?
            )
        )?
        (?P<dev>                                          # dev release
            [-_\.]?
            (?P<dev_l>dev)
            [-_\.]?
            (?P<dev_n>[0-9]+)?
        )?
    )
    (?:\+(?P<local>[a-z0-9]+(?:[-_\.][a-z0-9]+)*))?       # local version
"""

# Compile the regex pattern
_regex = re.compile(r"^\s*" + _VERSION_PATTERN + r"\s*$", re.VERBOSE | re.IGNORECASE)

# Test the compiled regex with some version strings
test_versions = [
    "v1.2.3",
    "1.2.3",
    "1.2.3-alpha1",
    "1.2.3-beta.2",
    "1.2.3-rc1",
    "1.2.3-post1",
    "1.2.3+local.version",
    "1!1.2.3",
    "1.2.3.dev1",
]

for version in test_versions:
    match = _regex.match(version)
    if match:
        print(f"Matched: {version}")
    else:
        print(f"Not matched: {version}")
```