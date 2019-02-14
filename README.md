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

**Converters**

Sometimes you have to convert the value of the option that user provided before using it. Converters are functions that are applied to certain options and replace their value in the Options object with the converted result of this function.

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

## CombinedOptions class

CombinedOptions is designed to merge several options dictionaries into one object. It is a common task when you have some global options set in foliant.yml but they can be overriden by tag options in Markdown source. The result is a dictionary-like CombinedOptions object which has all options from config and from the tag. Which option to use if they overlap is described by `priority` parameter.

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

Let's combine these two options in one object. To do this we will have to pack them into a single dictionary under arbitrary keys, and supply a priority string which should be one of aforementioned keys:

```python
>>> from foliant.preprocessors.utils.combined_options import CombinedOptions
>>> options = CombinedOptions({'config': config_options, 'tag': tag_options}, priority='tag')

```

Note that we've given tag_options a priority by supplying parameter `priority='tag'`

Now look at the values we are getting:

```python
>>> options['config']  # we have option from config_options
'config.xml'
>>> options['caption']  # we also have an option from tag_options
'Main screen'
>>> options['dpi']  # when we ask option which occurs in both, we get one from tag_options
500

```

Of course, CombinedOptions supports validation and convertors just as the Options class does.

You can also change the priority on fly. To do this just give a new value to the `priority` attribute:

```python
>>> options.priority = 'config'
>>> options['dpi']
300

```
