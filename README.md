[![](https://img.shields.io/pypi/v/foliantcontrib.utils.combined_options.svg)](https://pypi.org/project/foliantcontrib.utils.combined-options/) [![](https://img.shields.io/github/v/tag/foliant-docs/foliantcontrib.utils.combined_options.svg?label=GitHub)](https://github.com/foliant-docs/foliantcontrib.utils.combined_options)

# Overview

combined_options is a module which helps you cope with the options from foliant.yml and tag options.

Module has two classes:
- **Options** which extends functionality of an options dictionary,
- **CombinedOptions** which allows to combine config and tag options into one dictionary-like object.

# Usage

To use functions and classes from this module, install it with command

```bash
pip3 install foliantcontrib.utils.combinedoptions
```

Then in your preprocessor module import the Options or CombinedOptions class and wrap your options dictionaries in them:

```python
from foliant.preprocessors.utils.combined_options import CombinedOptions

...

options = CombinedOptions({'main': main_options,
                           'tag': tag_options},
                          priority='tag')
if 'caption' in options:
    self._caption = options['caption']
```

Options and CombinedOptions act like a dictionary. For detailed description of the functions, please refer to the rest of the documentation.

## Options class

Options class wraps around the options dictionary, for example from your foliant.yaml file, and gives it some extra functionality.

**Init parameters**

- `options` (dict, required) — the pure dictionary with options.
- `defaults` (dict, optional) — dictionary with default values, usually declared at the top of the preprocessor class.
- `convertors` (dict, optional) — dictionary with key = option name, value = convertor function which will be applied to the value of an option with such name before storing in class.
- `validators` (dict, optional) — dictionary with key = option name, value = validator function which will be applied to the value of this option. Function should check for validity and raise ValidationError if the check fails.
- `required` (list, optional) — a list of required parameters or a list of combinations of required parameters.

Let's say you have such options in your config:

```yaml
preprocessors:
    - MyAwesomePreprocessor:
        config: config.xml
        articles:
            - a1
            - a2
            - a3
        store_log: true
```

Foliant will parse this config into a dictionary which will look like this:

```python
>>> config_options = {'config': 'config.xml', 'articles': ['a1','a2','a3'], 'store_log': True}

```

Let's say you have a `defaults` dictionary in your preprocessor source code looking like this:

```python
>>> defaults = {'config': 'config.xml', 'articles': []}

```

Let's import the `Options` class to look at some of its functions:

```python
>>> from foliant.preprocessors.utils.combined_options import Options

```

To use the class we need to supply our options dictionary and the dictionary with default values to the class constructor:

```python
>>> options = Options(config_options, defaults)

```

> Note that supplying the dictionary with defaults is not required, it is needed only for the work of `is_default` class method

The resulting object acts just like a dictionary:

```python
>>> options['config']
'config.xml'
>>> 'articles' in options
True
>>> options.get('missing', 'value')
'value'

```

But now, since we've given it a dictionary with default values, we can check if the value set in options differs from its default:

```python
>>> options.is_default('config')
True
>>> options.is_default('articles')
False
>>> options.is_default('store_log')
False

```

Another function of this class is that it can validate option values and convert them.

Validators and convertors are functions which you'll have to create yourself. A few of them are already available in the module though, check the source code.

**Validators**

Validator is a function that takes option value as parameter and raises `ValidationError` it the value is wrong in some way.

For example, if you want to be sure that type the option user supplied is a string you can write a validator like this:

```python
>>> from foliant.preprocessors.utils.combined_options import ValidationError
>>> def validate_is_str(option):
...     if type(option) is not str:
...         raise ValidationError('Value should be string!')

```

To add validator to your options object, supply it in the constructor:

```python
>>> config_options = {'check': 123}
>>> options = Options (config_options, validators={'check': validate_is_str})
Traceback (most recent call last):
  ...
foliant.preprocessors.utils.combined_options.ValidationError: Error in option "check": Value should be string!

```

You see, it even didn't allow us to create an options object because the value of the parameter is wrong. You should handle this error on your own.

**Convertors**

Sometimes you have to convert the value of the option that user provided before using it. Convertors are functions that are applied to certain options and replace their value in the Options object with the converted result of this function.

For example, if we need a comma-separated string has to be converted into a list, we can write this kind of convertor:

```python
>>> def convert_to_list(option):
...     if type(option) is str:
...         return option.split(',')
...     else:
...         return option

```

So now let's attach our convertor to an option object:

```python
>>> config_options = {'names': 'Sam,Ben,Dan'}
>>> options = Options(config_options, convertors={'names': convert_to_list})
>>> options['names']
['Sam', 'Ben', 'Dan']

```

**Required options**

Options class may check if all of the required options are defined. To use this feature supply a list of required param names in the `required` key:

```python
>>> config_options = {'title': 'My article', 'id': 335}
>>> options = Options(config_options, required=['title', 'space'])
Traceback (most recent call last):
  ...
foliant.preprocessors.utils.combined_options.RequiredParamsMissingError: Not all required params are supplied: ['title', 'space']

```

We've forgot to define the required parameter `space` and Options object alerted us right away.

There are situations when you have not just required parameters but combination of required parameters. For example you can define the page you are editing by id, or by title and space. Both ways are possible, but one of them has to be satisfied.

In this case you can supply a list with all possible combination in the `required` key like this:

```python
>>> config_options = {'title': 'My article', 'id': 335}
>>> options = Options(config_options, required=[['title', 'space'], ['id']])

```

## CombinedOptions class

CombinedOptions is designed to merge several options dictionaries into one object. It is a common task when you have some global options set in foliant.yml but they can be overriden by tag options in Markdown source. The result is a dictionary-like CombinedOptions object which has all options from config and from the tag.

When options overlap, the priority would be given to the option from the dictionary, which defined it first. If you want to override this behavior (or make things more verbose) you can utilize the `priority` parameter.

CombinedOptions is inherited from Options class and repeats all its functionality.

**Init parameters**

- `options` (dict, required) — dictionary where key = priority, value = option dictionary.
- `priority` (str) — initial priority (if not set = first key from options dict).

Remaining parameters are the same as in Options class:

- `defaults` (dict, optional) — dictionary with default values, usually declared at the top of the preprocessor class.
- `convertors` (dict, optional) — dictionary with key = option name, value = convertor function which will be applied to the value of this option before storing in class.
- `validators` (dict, optional) — dictionary with key = option name, value = validator function which will be applied to the value of this option. Function should check for validity and raise ValidationError if the check fails.

To illustrate CombinedOptions' handiness let's assume that you have two option dictionaries, one came from foliant.yml and the other one — from the tag you are currently processing:

```python
>>> config_options = {'config': 'config.xml', 'dpi': 300}
>>> tag_options = {'dpi': 500, 'caption': 'Main screen'}

```

Let's combine these two options in one object. To do this we will have to pack them into a single dictionary under arbitrary keys (keys are explained later):

```python
>>> from foliant.preprocessors.utils.combined_options import CombinedOptions
>>> options = CombinedOptions({'tag': tag_options, 'config': config_options})

```

As you have noticed, we have parameter `'dpi'` defined in both option dictionaries. Let's look at the values we will be getting:

```python
>>> options['config']  # we have option from config_options
'config.xml'
>>> options['caption']  # we also have an option from tag_options
'Main screen'
>>> options['dpi']  # when we ask option which occurs in both, we get one from tag_options
500

```

In the CombinedOptions object we have options from both `config` and `tag` dictionaries. The conflicting options (like `dpi` in our example) are picked from the first dictionary defined. (If you are using Python 3.5 and older, supply options in `OrderedDict`)

If you wish to change the priority of the dictionaries (or make it more verbose), use the `priority` parameter, which may be a single string, o list of strings in priority order:

```python
>>> options = CombinedOptions({'tag': tag_options, 'config': config_options}, priority='config')
>>> options['dpi']  # now we should get the value from 'config' dictionary
300

```


You can also change the priority on fly. To do this just give a new value to the `priority` attribute:

```python
>>> options.priority = 'tag'
>>> options['dpi']
500

```

# Predefined convertors and validators

There are some convertors and validators already predefined in combined_options module.

**Validators**

`validate_in` — factory that returns a validator which checks if specified value is in the list.

To use this validator, first get one from the factory, supplying the list of correct values for option:

```python
>>> from foliant.preprocessors.utils.combined_options import validate_in
>>> correct = ['spam', 'eggs', 'bacon']
>>> validator = validate_in(correct)
>>> options = Options({'dish': 'chicken'}, validators={'dish': validator})
Traceback (most recent call last):
  ...
foliant.preprocessors.utils.combined_options.ValidationError: Error in option "dish": Unsupported option value chicken. Should be one of: spam, eggs, bacon

```

***

`val_type` — factory that returns validator, which checks if specified value is of correct type.

```python
>>> from foliant.preprocessors.utils.combined_options import val_type
>>> validator = val_type(str)
>>> options = Options({'name': 998}, validators={'name': validator})
Traceback (most recent call last):
  ...
foliant.preprocessors.utils.combined_options.ValidationError: Error in option "name": Unsupported option value 998. Must be of type <class 'str'>

```

You can also specify a list of supported types:

```python
>>> correct = [str, int, None]
>>> validator = val_type(correct)
>>> options = Options({'name': None}, validators={'name': validator})
>>> options = Options({'name': ['Bob', 'Alice']}, validators={'name': validator})
Traceback (most recent call last):
  ...
foliant.preprocessors.utils.combined_options.ValidationError: Error in option "name": Unsupported option value ['Bob', 'Alice']. Must be of type <class 'str'>, <class 'int'>, None

```

***

`validate_exists` — checks if specified path exists in file system.

```python
>>> from foliant.preprocessors.utils.combined_options import validate_exists
>>> options = Options({'fp': '/'}, validators={'fp': validate_exists})
>>> options = Options({'fp': '/wrong'}, validators={'fp': validate_exists})
Traceback (most recent call last):
  ...
foliant.preprocessors.utils.combined_options.ValidationError: Error in option "fp": Path /wrong does not exist.

```

**Convertors**

`yaml_to_dict_convertor` — converts yaml-string to python dict. If value is a dict already — just returns it. **DEPRECATED:** since Foliant 1.0.9 all tag option values are treated as YAML.

***

`boolean_convertor` — converts strings and integers into Boolean according to the table

value | result
----- | ------
`1` | `True`
`0` | `False`
`'1'` | `True`
`'0'` | `False`
`'y'` | `True`
`'n'` | `False`
`'yes'` | `True`
`'no'` | `False`
`'true'` | `True`
`'false'` | `False`
`<other>` | `True`

***

`rel_path_convertor` —  Convertor factory which makes path, supplied in option value, relative to parent_path, set during the convertor initialization. Returns `PosixPath` object.

```python
>>> from foliant.preprocessors.utils.combined_options import rel_path_convertor
>>> my_conv = rel_path_convertor('/usr/src/app')
>>> options = Options({'index': 'src/index.md'}, convertors={'index': my_conv})
>>> options['index']
PosixPath('/usr/src/app/src/index.md')

```
