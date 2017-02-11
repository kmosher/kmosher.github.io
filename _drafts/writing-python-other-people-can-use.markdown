---
title:  "Writing Python Scripts Other People Can Use"
layout: post
---

I love writing little python scripts. I _really, really_ love writing small python scripts. But there's a problem: **distributing small python scripts sucks**.

Previously, I've been living a bit of a fantasy life.
All my users were on Linux boxes that I had configuration management control over.
This meant I knew the exact environment I was targeting, and I could use [dh_virtualenv][dh_virtualenv] for all my packaging.

Now though I find myself targeting some less pleasent environments: 1) a bunch of user-managed OS X boxes and 2) an environment without a good CI pipline.
Both of these have sent me re-examining what the current state of the world is when it comes to turning small chunks of python code into small apps.

## Virtualenv

# Tools
[py2app]:             https://pythonhosted.org/py2app/index.html
[homebrew]:           http://brew.sh/
[py2exe]:             http://www.py2exe.org/
[cx_Freeze]:          https://cx-freeze.readthedocs.io/en/latest/
[pypa]:               https://packaging.python.org/
[pyInstaller]:        http://www.pyinstaller.org/
[dh_virtualenv]:      https://dh-virtualenv.readthedocs.io/en/1.0/usage.html
[pex]:                https://pex.readthedocs.io/en/stable/
[pipsi]:              https://github.com/mitsuhiko/pipsi
[PyRun]:              https://www.egenix.com/products/python/PyRun/
[virtualenv-burrito]: https://github.com/brainsik/virtualenv-burrito
[autoenv]:            https://github.com/kennethreitz/autoenv
# Good overviews
[glyph-talk]:         http://pyvideo.org/pycon-us-2016/glyph-shipping-software-to-users-with-python-pycon-2016.html
[glyph-blog]:         https://glyph.twistedmatrix.com/2015/09/software-you-can-use.html
[embedding-blog]:     http://hackerboss.com/how-to-distribute-commercial-python-applications/
[freezing-your-code]: http://docs.python-guide.org/en/latest/shipping/freezing/
