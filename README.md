# Pybind11 bindings for the Abseil C++ Common Libraries

These adapters make Abseil types work with Pybind11 bindings. For more
information on using Pybind11, see
g3doc/third_party/pybind11/google3_utils/README.md.

To use the converters listed below, just include the header
in the .cc file with your bindings:

```
#include "pybind11_abseil/absl_casters.h"
```

Support for non-const `absl::Span` for numeric types is also available by
including a separated header file:

```
#include "pybind11_abseil/absl_numpy_span_caster.h"
```

## absl::Duration

`absl::Duration` objects are converted to/ from python datetime.timedelta objects.
Therefore, C code cannot mutate any datetime.timedelta objects from python.

## absl::Time

`absl::Time` objects are converted to/from python datetime.datetime objects.
Additionally, datetime.date objects can be converted to `absl::Time` objects.
C code cannot mutate any datetime.datetime objects from python.

Python date objects effectively truncate the time to 0 (ie, midnight).
Python time objects are not supported because `absl::Time` would implicitly
assume a year, which could be confusing.

### Time zones
Python `datetime` objects include timezone information, while
`absl::Time` does not. When converting from Python to C++, if a timezone is
specified then it will be used to determine the `absl::Time` instant. If no
timezone is specified by the Python `datetime` object, the local timezone is
assumed.

When converting back from C++ to Python, the resultant time will be presented in
the local timezone and the `tzinfo` property set on the `datetime` object to
reflect that. This means that the caller may receive a datetime formatted
in a different timezone to the one they passed in. To handle this safely, the
caller should take care to check the `tzinfo` of any returned `datetime`s.

## absl::CivilTime
`absl::CivilTime` objects are converted to/from Python datetime.datetime
objects. Fractional Python datetime components are truncated when converting to
less granular C++ types, and time zone information is ignored.

## absl::Span

For non-const `absl::Span` and conversion from `numpy` arrays, see
[non-const absl::Span](#non-const-abslspan) later.

When `absl::Span<const T>` (i.e. the `const` version) is considered, there is
full support to mapping into Python sequences.
Currently, this will always result in the list being copied, so you lose the
efficiency gains of spans in native C++, but you still get the API versatility.

The value type in the span can be any type that pybind knows about. However, it
must be immutable (ie, `absl::Span<const ValueType>`). Theoretically mutable
ValueTypes could be supported, but with some subtle limitations, and this is
not needed right now, so the implementation has been deferred.

The `convert` and `return_value_policy` parameters will apply to the *elements*.
The list containing those elements will aways be converted/copied.

### non-const absl::Span
Support for non-cost `absl::Span`, for numeric types only, is provided for
`numpy` arrays. Support is only for output function parameters and not for
returned value. The rationale behind this decision is that, if a `absl::Span`
were to be returned, the C++ object would have needed to outlive the mapped
Python object. Given the complexity of memory management across languages, we
did not add support of returned `absl::Span`.
That is the following is supported:

```
void Foo(absl::Span<double> some_span);
```
while the following is not (it will generate a compile error):
```
absl::Span<double> Bar();
```

Note: It is possible to use the non-const `absl::Span` bindings to wrap a
function with `absl::Span<const T>` argument if you are using `numpy` arrays
and you do not want a copy to be performed. This can be done by defining a
lambda function in the `pybind11` wrapper, as in the following example. See
b/155596364 for more details.

```
void MyConstSpanFunction(absl::Span<const double> a_span);
...

PYBIND11_MODULE(bindings, m) {
  m.def(
      "wrap_span",
      [](absl::Span<double> span) {
        MyConstSpanFunction(span);
      });
}
```


## absl::string_view

Supported exactly the same way pybind11 supports `std::string_view`.

## absl::optional

Supported exactly the same way pybind11 supports `std::optional`.

## absl::flat_hash_map

Supported exactly the same way pybind11 supports `std::map`.

## absl::flat_hash_set

Supported exactly the same way pybind11 supports `std::set`.
