# pyp2spec

This project is a thought descendant of [specfile_generator](https://github.com/frenzymadness/specfile_generator).

It aims to generate working Fedora RPM spec file for Python projects.
The produced spec files must be compliant with the current [Python Packaging Guidelines](https://docs.fedoraproject.org/en-US/packaging-guidelines/Python/) (in effect since 2021).
It utilizes the benefits of [pyproject-rpm-macros](https://src.fedoraproject.org/rpms/pyproject-rpm-macros).

It is under development.

## What it does

`pyp2spec.py` gathers all the necessary information from PyPI to produce a valid
Fedora spec file and stores it in the current directory alongside with
the config file used to produce the spec file.

Inside, there are two parts:
- *pyp2conf*: gathers of all the necessary information to produce a spec file and stores it in a configuration file
- *conf2spec*: produces working spec file using all the information from configuration file

## How to run

To run whatever this project offers at this point, install to your virtual environment the dependencies from `requirements.txt`:

```
python -m pip install -r requirements.txt
```

To run the script and generate both the config and spec file, type:
```
python pyp2spec.py <pypi_package_name>
```

### Tests

Test framework used in the project is pytest.
The tests use pytest-regresssions extension used to generate spec files to compare.
The tests use betamax to record API calls and replay them from the prerecorded cassettes.
To run the tests, run following commands:

```
python -m pip install pytest pytest-regresssions betamax
python -m pytest -vv
```


## Configuration file specification

Configuration data is stored in a TOML file.

### Mandatory fields

- Generated by pyp2conf

| Field  | Description | Type |
| -------- | -------- | -------- |
| pypi_name | package name as stored in PyPI  | string   |
| python_name | pypi_name prepended with `python-` | string |
| archive_name | source tarball name, stripped of version and file extension  | string |
| version | package version to create spec file for  | string |
| release | Fedora package release | string |
| summary | short package summary | string |
| license | license name | string |
| url | project URL | string |
| source | tarball URL | string |
| description | long package description | multiline string |
| changelog_head | spec file changelog header (date, name, e-mail) | string |
| changelog_msg | spec file changelog message | string |
| modules | importable Python modules in the package | list |


### Optional fields

- Not generated by pyp2conf
- Can be also added to config files generated by pyp2conf


| Field | Description | Type |
| -------- | -------- | -------- |
| manual_build_requires     | additional build requires, eg. `python3-sphinx`     | list     |
| extra_build_requires     | extra options to %pyproject_buildrequires (`-t`, `-x`); default: runtime (`-r`)   | list     |
| extra_tox_env     | if `-x` is defined as extra, provide name of the tox env      | list     |
| test_method     | `pytest`, `tox`; default: `py3_check_import`     | string     |
| unwanted_tests     | test names to skip with pytest     | list     |
| binary_files     | list binary files from the package     | list     |
| license_files     | list license files from the package     | list     |
| doc_files     | list doc files from the package     | list |


### Example config file generated by pyp2spec

```
changelog_msg = "Package generated with pyp2spec"
changelog_head = "Tue Sep 21 2021 Name Surname <email@address>"
description = "This is package 'aionotion' generated automatically by pyp2spec."
summary = "A simple Python 3 library for Notion Home Monitoring"
version = "2.0.3"
license = "MIT"
release = "1"
pypi_name = "aionotion"
python_name = "python-aionotion"
modules = [
    "aionotion",
]
url = "https://github.com/bachya/aionotion"
source = "%{pypi_source aionotion}"
archive_name = "aionotion"
```

### Spec file generated using the example config

```
Name:           python-aionotion
Version:        3.0.2
Release:        1%{?dist}
Summary:        A simple Python 3 library for Notion Home Monitoring

# Check if the automatically generated License and its spelling is correct for Fedora
# https://docs.fedoraproject.org/en-US/packaging-guidelines/LicensingGuidelines/
License:        MIT
URL:            https://github.com/bachya/aionotion
Source0:        %{pypi_source aionotion}

BuildArch:      noarch
BuildRequires:  python3-devel

%global _description %{expand:
This is package 'aionotion' generated automatically by pyp2spec.}


%description %_description

%package -n     python3-aionotion
Summary:        %{summary}

%description -n python3-aionotion %_description


%prep
%autosetup -p1 -n aionotion-%{version}


%generate_buildrequires
%pyproject_buildrequires -r


%build
%pyproject_wheel


%install
%pyproject_install
%pyproject_save_files '*' +auto


%check
%py3_check_import aionotion


%files -n python3-aionotion -f %{pyproject_files}


%changelog
* Tue Sep 21 2021 Name Surname <email@address> - 3.0.2-1
- Package generated with pyp2spec
```


## License

The code is licensed under **MIT**.

The spec file template - `template.spec` and the filed generated by the tool are licensed under **CC0 1.0 Universal**.