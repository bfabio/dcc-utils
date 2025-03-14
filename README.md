# DCC Utils
Python library to decode the EU Covid-19 vaccine certificate, as [specified by the EU](https://ec.europa.eu/health/ehealth/covid-19_en).

[![Latest Version](https://img.shields.io/pypi/v/dcc-utils.svg)](https://pypi.python.org/pypi/dcc-utils/)
[![CI](https://github.com/astagi/dcc-utils/actions/workflows/ci.yml/badge.svg)](https://github.com/astagi/dcc-utils)
[![Coverage](https://codecov.io/gh/astagi/dcc-utils/branch/master/graph/badge.svg?token=SZ7lyP073V)](https://codecov.io/gh/astagi/dcc-utils)
[![Supported Python versions](https://img.shields.io/badge/python-3.7%2C%203.8%2C%203.9%2C%203.10-blue.svg)](https://pypi.python.org/pypi/dcc-utils/)
[![License](https://img.shields.io/github/license/astagi/dcc-utils.svg)](https://pypi.python.org/pypi/dcc-utils/)
[![Downloads](https://img.shields.io/pypi/dm/dcc-utils.svg)](https://pypi.python.org/pypi/dcc-utils/)


## Setup

```sh
pip install dcc-utils
```

Make sure `zbar` is installed in your system
  * For Mac OS X, it can be installed via `brew install zbar`
  * Debian systems via `apt install libzbar0`. [Source](https://pypi.org/project/pyzbar/)
  * Fedora / Red Hat `dnf install zbar`

## Usage

### Parse DCC

This library takes an image with a QR code or a raw repr. of a vaccine certificate as
the parameter and will show the certificate's content. 

```py
from dcc_utils import dcc

dcc_from_img = dcc.from_image("/my/certificate/path")
dcc_from_raw = dcc.from_raw("HC1:6BF...FTPQ3C3F")
```

Then you can access to `payload` and `kid`

```py
assert dcc_from_img.kid == "53FOjX/4aJs="
assert dcc_from_img.payload["v"][0]["ci"] == "URN:UVCI:01:FR:W7V2BE46QSBJ#L"
```

👉🏻 `payload` follows [EU Digital COVID Certificates JSON Schema Specification](https://ec.europa.eu/health/sites/default/files/ehealth/docs/covid-certificate_json_specification_en.pdf)

`from_image` and `from_raw` methods may rise `DCCParsingError`

```py
from dcc_utils.exceptions import DCCParsingError
```

### Validate DCC digital signature

```py
signature = b"""
-----BEGIN CERTIFICATE-----
MIIIAjCCBeqgAwIBAgIQAnq8g/T
-----END CERTIFICATE-----
"""
assert my_dcc.check_signature(signature)
```

`check_signature` method may rise `DCCSignatureError`

```py
from dcc_utils.exceptions import DCCSignatureError
```

### Evaluate CertLogic business rules

With dcc-utils you can evaluate [business rules](https://github.com/ehn-dcc-development/dgc-business-rules) against a DCC

```py
from dcc_utils import rule, dcc

my_dcc = dcc.from_image("/my/certificate/path")
my_rule = rule.from_file("/my/rule.json")
print(my_rule.description["en"])
my_rule.evaluate_dcc(my_dcc) # True or False
```

`evaluate_dcc` accepts extra variables as a second parameter, e.g. `validationClock`

```py
import datetime
clock = datetime.datetime(2022, 10, 10, 0, 0, tzinfo=datetime.timezone.utc)
my_rule.evaluate_dcc(
    my_dcc,
    {
        "validationClock": clock,
    },
)
```

you can also load rules from JSON (`from_json`), useful to evaluate rules 
exposed on a server

```py
my_rule = rule.from_json({...})
```

`from_file` and `from_json` method may rise `DCCRuleError`

```py
from dcc_utils.exceptions import DCCRuleError
```

## Dev setup

Install dependencies using pip:

```
pip install -r requirements-dev.txt
```

Run tests

```
make test
``` 

## EU Digital COVID Certificate Specifications
What's in a EU Digital COVID/Green Certificate?
* Value Sets for Digital Green Certificates https://ec.europa.eu/health/sites/default/files/ehealth/docs/digital-green-certificates_dt-specifications_en.pdf
* JSON schema: https://github.com/ehn-dcc-development/ehn-dcc-schema

### Sample data
Digital Green Certificate Gateway (DGCG) samples for all participating countries:
https://github.com/eu-digital-green-certificates/dgc-testdata

## Credits
Parts of this code are adapted from [vacdec project](https://github.com/HQJaTu/vacdec).

## License
This library is available under the [MIT](https://opensource.org/licenses/mit-license.php) license.
