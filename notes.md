# config.py
**导入的包**
```python
import argparse # 
import os
import genotypes as gt
from functools import partial
import torch
```
1. 通过使用argparse，您可以轻松地定义和解析命令行参数，使您的程序具备更好的用户交互性和易用性。它在开发命令行工具、脚本和应用程序时非常有用，并广泛应用于Python项目中。
2. 提供了一种创建偏函数（Partial Function）的方式。偏函数是指固定一个函数的某些参数，然后返回一个新的函数，该新函数可以接受剩余的参数并执行原始函数。换句话说，偏函数可以创建一个带有预设参数的新函数。
partial函数的功能包括：

 - 固定函数的部分参数：通过partial函数，您可以选择性地固定函数的一部分参数，将它们预设为特定的值。
 - 创建新的可调用对象：partial函数会返回一个新的可调用对象，它具有原始函数的功能，但具有部分参数已经被预设的特性。
 - 接受剩余的参数：通过使用偏函数创建的新函数，您可以将剩余的参数传递给它，这些参数将与预设的参数一起传递给原始函数进行执行。

 &emsp;&emsp;使用偏函数的好处是可以简化函数调用，减少重复代码，并提高代码的可读性。您可以根据需要选择性地固定函数的参数，以便在后续的调用中更方便地使用。
 
 ```python
 def get_parser(name):   # 用于创建一个默认格式的解析器对象
    """ make default formatted parser """
    parser = argparse.ArgumentParser(name, formatter_class=argparse.ArgumentDefaultsHelpFormatter)  # argument parser 参数解析器 名称设置为name
    # 同时指定了帮助信息的格式化类为argparse.ArgumentDefaultsHelpFormatter，以显示参数的默认值。
    # print default value always
    parser.add_argument = partial(parser.add_argument, help=' ')  # 通过这种方式，我们可以更简洁地添加命令行参数，并且所有添加的参数都具有相同的默认帮助信息（空字符串）
    # 通过使用partial函数对parser.add_argument方法进行包装，将其设置为一个默认添加帮助信息的方法，即所有添加的参数都具有相同的默认帮助信息（空字符串）。
    return parser   # 返回创建的解析器对象
```
