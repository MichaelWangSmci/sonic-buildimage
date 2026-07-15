# AGENTS.md

## Cursor Cloud specific instructions

### What this repo is
`sonic-buildimage` is the build system for SONiC (a network operating system). A
**full** product build produces an ONIE/Aboot switch OS image and is done
**inside Docker "sonic-slave" containers** via `make init` / `make configure
PLATFORM=<vendor>` / `make all`. That flow needs Docker + KVM, hundreds of GB of
disk, and hours of compilation, so it is **not** the practical dev loop in the
Cloud VM (Docker is not installed here). Most day-to-day work on the C++/Python
daemons also happens inside that slave container because they link against
SONiC-specific binaries (e.g. `swsscommon`, built from the `sonic-swss-common`
submodule) that are **not** on PyPI.

### What runs standalone in this VM (the supported dev loop here)
The in-repo Python package **`src/sonic-yang-models`** (SONiC's YANG config
schemas) and its companion **`src/sonic-yang-mgmt`** validate config against the
schema and have **no `swsscommon` dependency**, so they run directly in a venv.
This is the component set that is set up and verified here.

Packages that import `swsscommon` (e.g. `sonic-py-common`, `sonic-config-engine`
/ `sonic-cfggen`, `sonic-dhcp-utilities`, `sonic-bgpcfgd`, ...) cannot be unit
tested outside the slave container without first building `swsscommon`.

### Environment layout (prebuilt in the VM snapshot)
- Python venv at **`/workspace/.venv`** (git-ignored via `.git/info/exclude`).
  The update script recreates/refreshes it idempotently with the standard PyPI
  deps. Run tools via `/workspace/.venv/bin/python` (or put `.venv/bin` on PATH).
- **libyang C 3.12.2** built from source and installed to `/usr/local` (Ubuntu
  apt only ships libyang v2, which is too old for the SONiC models). `ldconfig`
  already knows `/usr/local/lib`.
- **libyang-python 3.1.0** built into the venv with SONiC's patches applied
  (notably `json_string_datatypes`, from `src/libyang3-py3/patch/`). The stock
  PyPI `libyang` will NOT work: 5.x needs libyang C v5, and the unpatched 3.1.0
  lacks the `json_string_datatypes` kwarg the tests use.

These native pieces are **not** reinstalled by the update script (they persist in
the snapshot). If the venv is ever wiped, rebuild the binding with:
`sudo apt-get install -y libyang2-dev cmake libpcre2-dev python3-dev` (or rebuild
libyang C 3.12.2 from `https://github.com/CESNET/libyang.git` tag `v3.12.2` into
`/usr/local`), then clone `CESNET/libyang-python` tag `v3.1.0`, apply patches
`0002/0003/0004` from `src/libyang3-py3/patch/` with `patch -p1`, and
`PKG_CONFIG_PATH=/usr/local/lib/pkgconfig pip install <that dir>`.

### YANG models are generated from templates
`src/sonic-yang-models/yang-models/*.yang` includes files rendered from
`yang-templates/*.yang.j2` (e.g. `sonic-types.yang`). You must build once so the
templates render, otherwise model loading fails with "sonic-types not found":
```
cd src/sonic-yang-models && /workspace/.venv/bin/python setup.py build
```

### Test / lint / validate (sonic-yang-models)
```
cd /workspace/src/sonic-yang-models
# YANG schema-validation suite (libyang) + legacy pyang model-tree gate:
PATH=/workspace/.venv/bin:$PATH /workspace/.venv/bin/python -m pytest \
    tests/test_sonic_yang_models.py tests/yang_model_pytests -q   # 929 pass
# Python lint (critical errors) on the active suite:
/workspace/.venv/bin/flake8 --select=E9,F63,F7,F82 \
    tests/yang_model_pytests tests/test_sonic_yang_models.py setup.py
```
Notes:
- The legacy `tests/test_sonic_yang_models.py` shells out to `pyang`, so
  `.venv/bin` must be on `PATH`.
- The older `tests/yang_model_tests/test_yang_model.py` has pre-existing
  `flake8 F821` findings; the active suite above is `tests/yang_model_pytests`.
