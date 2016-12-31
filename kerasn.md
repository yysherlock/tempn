#Kera Notes

----------
> * Layers
> * Models

## Examples
* question-answering with memory networks
* text generation with stacked LSTMs
* [Phased LSTM][6]
## Quick Example
<dot>
digraph a {
    b -> c -> d
}
</dot>

## keras.layers
### 1. Dense Layer
* Constructors
> * Dense(output_dim, input_dim=N)
> * Dense(output_dim, input_shape=(N,))
```python
    # as first layer in a sequential model:
    model = Sequential()
    model.add(Dense(32, input_dim=16))
    # now the model will take as input arrays of shape (*, 16)
    # and output arrays of shape (*, 32)

    # this is equivalent to the above:
    model = Sequential()
    model.add(Dense(32, input_shape=(16,)))

    # after the first layer, you don't need to specify
    # the size of the input anymore:
    model.add(Dense(32))
```

### 2. Activation Layer
* Constructor
> * Activation(activation_function_name)
     activation_function_name: softmax, elu, softplus, softsign, relu, tanh, sigmoid, hard_sigmoid, linear
```python
Activation("relu")
```
- Learn `get_from_model` in activations.py, utils/generic_utils.py
```python
# activations.py
def elu(...): pass
def relu(...): pass
def sigmoid(...): pass
def linear(...): pass

def get(identifier):
    if identifier is None:
        return linear
    return get_from_module(identifier, globals(), 'activation function')
```
> - globals(), [built-in function][1], return a dict representing the current global symbol table. 
This is always the dictionary of the current module (inside a function or method, this is the module where it is defined, not the module from which it is called).
``` python
# utils/generic_utils.py
def get_from_module(identifier, module_params, module_name, instantiate=False, kwargs=None):
    if isinstance(identifier, six.string_types): # 如果 identifier 是字符串类型
        result = module_params.get(identifier) # default_value is `None`
        if not result:
            raise ValueError('Invalid ' + str(module_name) + ':' + str(identifier))
        if instantiate and not kwargs:
            return result()
        elif instantiate and kwargs:
            return result(**kwargs)
        else:
            return result
    elif isinstance(identifier, dict): # 如果 identifier 是字典类型
     # identifier应是这样的字典,存放了激励函数名字与该激励函数所需的参数: {'name':activation_function_name, arg1:value1, ...} 
        name = identifier.pop('name')
        result = module_params.get(name)
        if result:
            return result(**identifier)
        else:
            raise ValueError('Invalid ' + str(module_name) + ':' + str(identifier))
```
 > - [`six`, python package][2]. It provides simple utilities for wrapping over differences between Python2 and Python3. It is intended to support codebases that work on both Python2 and Python3 without modification.  six consists of only one Python file, so it is painless to copy into a project.
 six.PY2, six.PY3
A boolean indicating if the code is running on Python2/3.
> - six.string_types
Possible types for text data. This is `basestring()` in Python2 and `str` in Python3. 
 Six provides constants that may differ between Python versions. Ones ending `_types` are mostly useful as the second argument to [`isinstance`][3] or `issubclass`.
> - [`d.get(key[,default_value])`][4], never raises a KeyError.
>- [`d.pop(key[,default_value])`][5], remove the key and return its value.

>>> 


[1]: https://docs.python.org/3/library/functions.html
[2]: http://pythonhosted.org/six/
[3]: https://docs.python.org/3/library/functions.html#isinstance
[4]: https://docs.python.org/3/library/stdtypes.html?highlight=dictionary#dict.get
[5]: https://docs.python.org/3/library/stdtypes.html?highlight=dictionary#dict.pop
[6]: https://github.com/dannyneil/public_plstm