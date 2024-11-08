---
title: Observables
date: 2024-11-08T08:20:00Z
draft: false
cascade:
  type: docs
---

# Usage

To ease observables creation, new functions have been implemented in `observable` module:

```python
>>> from core.schemas import observable
>>> observable.guess_type("192.168.1[.]2")
'ipv4'
>>> observable.create(value="192.168.1[.]2")
IPv4(value='192.168.1.2', created=datetime.datetime(2024, 10, 29, 23, 57, 4, 504655, tzinfo=datetime.timezone.utc), context=[], last_analysis={}, type='ipv4', id=None, tags={}, root_type='observable')
>>> observable.find(value="192.168.1[.]2")
>>> observable.save(value="192.168.1[.]2")
IPv4(value='192.168.1.2', created=datetime.datetime(2024, 10, 29, 23, 57, 20, 82795, tzinfo=TzInfo(UTC)), context=[], last_analysis={}, type='ipv4', id='5638348', tags={}, root_type='observable')
>>> observable.find(value="192.168.1[.]2")
IPv4(value='192.168.1.2', created=datetime.datetime(2024, 10, 29, 23, 57, 20, 82795, tzinfo=TzInfo(UTC)), context=[], last_analysis={}, type='ipv4', id='5638348', tags={}, root_type='observable')
```

## Guess observable type from value

```python
guess_type(value: str)
``` 

Guess which observable type can validate `value` string. If the value can be validated by an observable, its type is returned as a string. If the value can't be guessed, `None` is returned.

## From string

### Create

Create an observable from the given string value without saving it in the database. It's up to the caller to then call `save()` on the returned object to save it in the database.

```python
create(value: str, type: str | None = None, **kwargs)
```

* Params
   * `value`: a string defining the observable.
   * `type`: an optional string defining the type of the observable. If `type` is not provided, it will be guessed by calling `guess_type`.
   * `kwargs`: a dictionary defining further attributes depending on the type of the observable.

* Returns
If the function call succeeds, the created observable is returned. Otherwise, an `ValueError` exception is raised.

### Save

Use `create` function and save observable in the database. Tag the resulting observable with the list of tags if defined. 

```python
save(*, value: str, type: str | None = None, tags: List[str] = None, **kwargs) -> ObservableTypes
```

* Params
   * `value`: a string defining the observable.
   * `type`: an optional string defining the type of the observable. If `type` is not provided, it will be guessed by calling `guess_type`.
   * `tags`: an optional list of strings corresponding to the tags to add to the saved observable.
   * `kwargs`: a dictionary defining further attributes depending on the type of the observable.

* Returns
If the function call succeeds, the saved observable is returned. Otherwise, an `ValueError` exception is raised.

## From text

### Create

Create observables from the given text. Each line must contain one observable. Guess the type of the observable and create it. Created observable are not saved in the database.

```python
create_from_text(text: str)
```

* Params
   * `text`: a string containing one observable per line.
* Returns
A tuple containing a list of created observables and a list of string for unguessable lines.

### Save

Use `create_from_text` function and save observables in the database. Tag the resulting observables with the list of tags if defined. 

```python
save_from_text(text: str, tags: List[str] = None)
```

* Params
   * `text`: a string containing one observable per line.
   * `tags`: an optional list of strings corresponding to the tags to add to the saved observables.
* Returns
A tuple containing a list of saved observables and a list of string for unguessable lines.

## From file

### Create

Create observables from the given file. Each line must contain one observable. Guess the type of the observable and creates it. Created observables are not saved in the database.

```python
create_from_file(file: FileLikeObject)
```

* Params
   * `file`: Represents a file in different ways:
      * str, bytes or os.PathLike: the provided file path will be opened for reading lines
      * io.IOBase: any file object either created from `open` or based on `io` classes
      * tempfile.SpooledTemporaryFile: defined to support API endpoint from UploadFile FastAPI.
* Returns
A tuple containing a list of created observables and a list of string for unguessable lines.

### Save

Use `create_from_file` function and save observables into the database. Tag the resulting observables with the list of tags if defined. 

```python
save_from_file(file: FileLikeObject, tags: List[str] = None)
```

* Params
   * `file`: Represents a file in different ways:
      * str, bytes or os.PathLike: the provided file path will be opened for reading lines
      * io.IOBase: any file object either created from `open` or based on `io` classes
      * tempfile.SpooledTemporaryFile: defined to support API endpoint from UploadFile FastAPI.
   * `tags`: an optional list of strings corresponding to the tags to add to the saved observables.
* Returns
A tuple containing a list of saved observables and a list of string for unguessable lines.

## From a url

### Create

Create observables from the given url. Each line must contain one observable. Guess the type of the observable and create it. Created observables are not saved in the database.

```python
create_from_url(url: str)
```

* Params
   * `url`: a string defining the URL to fetch the content from.
* Returns
A tuple containing a list of created observables and a list of string for unguessable lines.

### Save

Use `create_from_url` function and save observables in the database. Tag the resulting observabls with the list of tags if defined. 

```python
save_from_url(url: str, tags: List[str] = None)
```

* Params
   * `url`: a string defining the URL to fetch the content from.
   * `tags`: an optional list of strings corresponding to the tags to add to the saved observables.
* Returns
A tuple containing a list of saved observables and a list of string for unguessable lines.

## Find an observable object

```python
find(value: str, **kwargs)
```

Find an observable in the database matching the string `value` and optional fields represented by `kwargs`. This function automatically refangs defanged `value` before querying the database. Return an observable object.

## Validators

### Overview

Validators are enforced for observables listed below. Validators ensure that provided value is relevant and that an for example an `IPv4` observable cannot be built from a `Url`.

* bic
* email
* hostname
* iban
* ipv4
* ipv6
* mac_address
* md5
* path
* sha1
* sha256
* url

{{< callout type="info" >}}
Validators enforcement also impacts `url` observable creation which requires a scheme to be valid. This means that `example.com/foobar` is no longer a valid `url`.  
{{< /callout >}}

### Implementation details

Value validation relies on pydandic `field_validator` decorator. Any observable implementation that requires validation must implement `validate_value` class method with `@field_validator("value")` decorator. `validate_value` is also where refang of the value takes place. If the value is not valid, ValueError exception must be raised. Otherwise, the value (modified or not) must be returned as a string.

```python
class IPv4(observable.Observable):
    type: Literal["ipv4"] = "ipv4"

    @field_validator("value")
    @classmethod
    def validate_value(cls, value: str) -> str:
        value = observable.refang(value)
        if not validators.ipv4(value):
            raise ValueError("Invalid ipv4 address")
        return value
```
