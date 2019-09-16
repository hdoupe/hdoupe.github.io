# ParamTools 0.10.0



**Highlights:**

- Better data access and CSV/spreadsheet compatitbility

- Performance improvements
- Ability to specify ParamTools operators like `label_to_extend` from the `defaults` object
- Bug fix for handling array parameters

### Better data access and CSV/spreadsheet compatibility

Many projects and their users are more familiar with spreadsheets than JSON files. Although ParamTools does not have native support for spreadsheets, it tries to make writing [custom `adjust` methods](https://paramtools.org/api/custom-adjust/) for this type of use case as ergonomic as possible. The example in the custom adjustment docs shows how to utilize this flexibility to read in CSV files and convert them to ParamTools adjustments. Tax-Cruncher, an open-source tax-policy model, used this approach to allow its users to adjust the default parameters with CSV files. This real-world example showed that ParamTools needed to make it easier for projects to loop over their parameters and their values. This release adds a few methods like `.items()` and `.keys()` that make `Parameters` objects feel more like Python dictionaries, yielding a familiar, intuitive interface:

```python
for param in params:
    print(param)

# one_param
# two_param

for param, value in params.items():
    print(param, value)

# one_param [{'value': 'hello'}]
# two_param [{'value': 'world'}]

params.keys()

# odict_keys(['one_param', 'two_param'])

params.to_dict()

# {
#     "one_param": [{"value": "hello"}],
#     "two_param": [{"value": "world"}]
# }
```

An unexpected advantage of adding these methods is that you can create a pandas `DataFrame` directly from a `Parameters` instance:

```python
import pandas as pd
import paramtools

class Params(paramtools.Parameters):
    defaults = {
        "a": {
            "title": "A",
            "description": "",
            "type": "int",
            "number_dims": 1,
            "value": [0]
        },
        "b": {
            "title": "B",
            "description": "",
            "type": "int",
            "number_dims": 1,
            "value": [0]
        }
    }
    array_first = True  # values stored as numpy arrays by default


params = Params()

params.adjust({
    "a": [1, 2, 3, 4, 5],
    "b": [6, 7, 8, 9, 10]
})


params_df = pd.DataFrame.from_dict(
    params.to_dict()
)

print(params_df)

#    a   b
# 0  1   6
# 1  2   7
# 2  3   8
# 3  4   9
# 4  5  10

```

### Performance Improvements

ParamTools 0.10.0 brings significant performance improvements for parameter lookups and updates. Prior to this release, ParamTools used a simple but naive approach for searching over its low-level data structure: a list of dictionaries. For updates, the logic looked like this:

```python
# loop over all new values
for i in range(len(new_values)):
    curr_vals = self._data[param]["value"]
    matched_at_least_once = False
    labels_to_check = tuple(k for k in new_values[i] if k != "value")
    to_delete = []
    # for each loop over the new values, loop over the existing values
    # to look for matches.
    for j in range(len(curr_vals)):
        # in our THIRD nested for loop, loop over all of the labels
        # in the new values.
        for label in labels_to_check:
            if curr_vals[j][label] == new_values[i][label]:
                match = True
                break
        if match:
            matched_at_least_once = True
            curr_vals[j]["value"] = new_values[i]["value"]
    if not matched_at_least_once:
        curr_vals.append(new_values[i])
```

The search function used a similar algorithm.  The new approach sorts the incoming value objects and the existing value objects by their labels and their labels' values. It then does a single loop over the labels and keeps track of the matches between the incoming value objects and the existing value objects. Not only is there less algorithmic complexity, but much of the computational work utilizes Python's `sets` data structure and its fast union and intersection operations. The result is a speed up of 5-14x for updates and 2-4x on search operations. I detailed the profiling approach and results in PR [#74](https://github.com/PSLmodels/ParamTools/pull/74), and I plan to do a more formal write-up in a follow-up post. Until then, the [`tree.py`](https://github.com/PSLmodels/ParamTools/blob/master/paramtools/tree.py) module (and its extensive doc strings and comments) is the best source of documentation for the new implementation.

### Add configuration to the `defaults.json` file

The "schema" object in the `defaults.json` file now includes an "operators" object for specifying the value for attributes like `label_to_extend` or `array_first`. A `dump` method is added for outputting a `Parameters` instance's data to a JSON sting. This makes it easier for a webapp like Compute Studio to consume a project's ParamTools configuration even if the project itself is not installed on the webserver.

```JSON
defaults = {
    "schema": {
        "operators": {
            "array_first": true,
            "label_to_extend": "year"
        },
    },
    "standard_deduction": {
        "title": "Standard deduction amount",
        "description": "Amount filing unit can use as a standard deduction.",
        "type": "float",
        "value": 10000
    },
}
```



### Bug fixes

- This release fixes a bug where array parameters were not handled correctly when `array_first` is `True`. ([PR #76](https://github.com/PSLmodels/ParamTools/pull/76))

