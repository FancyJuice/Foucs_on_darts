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
```python
def parse_gpus(gpus):
    if gpus == 'all':
        return list(range(torch.cuda.device_count()))
    else:
        return [int(s) for s in gpus.split(',')]
# torch.cuda.device_count()返回当前系统中可用的GPU设备数量。再生成序列，再生成列表。
# gpus 的输入类型为字符串，由gpu的id组成，由逗号分隔 else之后的语句为返回该id整数型的参数列表
```
```python
class BaseConfig(argparse.Namespace):   # 继承自argparse.Namespace。该类提供了一些方法来打印参数和将参数转换为Markdown格式。
    # argparse.Namespace是argparse模块中的一个类，用于创建命名空间对象。命名空间对象是一个简单的容器，可以用来保存命令行解析的结果。
    #
    # 当使用argparse解析命令行参数时，它将解析的结果存储在argparse.Namespace对象中。该对象的属性将对应命令行参数的名称，并且其值将对应相应的参数值。
    #
    # argparse.Namespace类没有定义任何方法，它只是一个简单的数据容器类。您可以通过访问属性来访问和操作命名空间对象中的值。
    # argparse.Namespace类实际上是一个简单的类字典对象，因此您还可以使用类字典的方法（如keys()、values()、items()等）来访问和操作命名空间对象中的值。
    def print_params(self, prtf=print):
        prtf("")    # 插入空行
        prtf("Parameters:")
        for attr, value in sorted(vars(self).items()):
            # vars(self)函数返回一个类实例的属性字典，其中属性名是键，对应的属性值是值。它等效于self.__dict__。
            # items()方法用于返回属性字典中的键值对作为元组的列表。
            # sorted()函数对属性字典中的键值对列表进行排序，按属性名称的字母顺序进行升序排序。
            # 因此，sorted(vars(self).items())返回一个按属性名称排序的属性键值对的列表。
            prtf("{}={}".format(attr.upper(), value))
            # {} 是一个占位符，用于表示待填充的位置。在这个例子中，它表示第一个占位符应该被替换为属性的名称，第二个占位符应该被替换为属性的值。
            # format() 方法是一个字符串的方法，用于将参数值填充到占位符的位置。
        prtf("")

    def as_markdown(self):
        """ Return configs as markdown format """
        text = "|name|value|  \n|-|-|  \n"
        for attr, value in sorted(vars(self).items()):
            text += "|{}|{}|  \n".format(attr, value)

        return text
```

```python
class SearchConfig(BaseConfig):
    def build_parser(self):
        parser = get_parser("Search config")
        parser.add_argument('--name', required=True)    # required=True表示在解析命令行参数时，必须提供
        parser.add_argument('--dataset', required=True, help='CIFAR10 / MNIST / FashionMNIST')
        parser.add_argument('--batch_size', type=int, default=64, help='batch size')    # 参数的类型为整数,参数的默认值为64
        parser.add_argument('--w_lr', type=float, default=0.025, help='lr for weights')     # 权重的学习率
        parser.add_argument('--w_lr_min', type=float, default=0.001, help='minimum lr for weights')     # 权重的学习率的最小值
        parser.add_argument('--w_momentum', type=float, default=0.9, help='momentum for weights')       # 权重的动量
        parser.add_argument('--w_weight_decay', type=float, default=3e-4,
                            help='weight decay for weights')
        parser.add_argument('--w_grad_clip', type=float, default=5.,
                            help='gradient clipping for weights')
        parser.add_argument('--print_freq', type=int, default=50, help='print frequency')
        parser.add_argument('--gpus', default='0', help='gpu device ids separated by comma. '
                            '`all` indicates use all gpus.')
        parser.add_argument('--epochs', type=int, default=50, help='# of training epochs')
        parser.add_argument('--init_channels', type=int, default=16)
        parser.add_argument('--layers', type=int, default=8, help='# of layers')
        parser.add_argument('--seed', type=int, default=2, help='random seed')
        parser.add_argument('--workers', type=int, default=4, help='# of workers')
        parser.add_argument('--alpha_lr', type=float, default=3e-4, help='lr for alpha')
        parser.add_argument('--alpha_weight_decay', type=float, default=1e-3,
                            help='weight decay for alpha')

        return parser

    def __init__(self):
        parser = self.build_parser()
        args = parser.parse_args()  # 解析命令行参数，并将解析结果存储在args变量中。args是一个命名空间（Namespace）对象，它包含了命令行参数的值。
        super().__init__(**vars(args))      # 调用了父类（基类）的构造函数，通过**vars(args)将命令行参数的值作为关键字参数传递给构造函数。
        # vars(args)将args对象转换为字典，其中字典的键是命令行参数的名称，值是命令行参数的值。通过使用**运算符，可以将字典中的键值对展开为关键字参数传递给构造函数。

        self.data_path = './data/'
        self.path = os.path.join('searchs', self.name)      # 将两个路径拼接成一个新的路径
        self.plot_path = os.path.join(self.path, 'plots')
        self.gpus = parse_gpus(self.gpus)   # 为gpu的id的整数列表
```
