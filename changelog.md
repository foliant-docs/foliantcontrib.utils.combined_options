# 1.0.10

- Fix: default dict was overriden after the first use.
- Allow to supply the list of priorities instead of just one priority.
- Priority for those option dictionaries, which are not mentioned in the `priority` param, are now defined by the order dictionaries are defined.

# 1.0.9

- defaults now actually supply default value (before they were only used for validation)
- add None to possible type validation in `val_type`.

# 1.0.8

- Fix `validate_exists` validator.

# 1.0.7

- Add `validate_exists` validator.
- Add `rel_path_convertor`.

# 1.0.6

- Add check for required params.
- Add val_type validator to check for param types.
- Allow to set values in Options objects

# 1.0.5

- support PyYAML 5.1

# 1.0.4

- Add path_convertor which converts string options to pathlib.PosixPath

# 1.0.3

- add __iter__ method to allow `for param in options`

# 1.0.2

- add '0'\'1' to bool_convertor

# 1.0.1

- Add boolean convertor

# 1.0.0

-   Initial release.
