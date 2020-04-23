---
title:  "Python Asterisk"
toc: true
# toc_label: "Table of Contents"
toc_stick: true

categories:
  - Python
tags:
  - Language
---

### Python Asterisk

python 에서는 *를 자주 활용하게 됩니다. C나 C++을 배우신 분들은 *를 보면 포인터가 생각나실 수 있지만, python에서 *는 포인터를 의미하지 않습니다. unpacking이라고 해서 packing된 인자를 unpack 하는 데 사용됩니다.


#### Positional Argument

아래 예제는 함수 인자가 가변인자일 때 그 가변 인자들을 tuple로 받는 예제입니다.

***args 를 활용한 예제 - 1**
```python
def print_item(first, second, *args):
    print(first)
    print(second)
    print(*args)

if __name__ == '__main__':
    print_item('first_item', 'second_item', 'third_item')
    print('========================')
    print_item('first_item', 'second_item', 'third_item', 'forth_item')
    print('========================')
    print_item('first_item', 'second_item', 'third_item', 'forth_item', 'fifth_item')
```
**출력**
```
first_item
second_item
third_item
========================
first_item
second_item
third_item forth_item
========================
first_item
second_item
third_item forth_item fifth_item
```

출력을 보면, *args에 입력된 값들이 `third_item forth_item fifth_item`라는 것을 확인할 수 있습니다.

다음은 unpacking을 하지 않고 출력을 시도했을 때의 예제입니다.

***args 를 활용한 예제 - unpacking 을 하지 않고 출력**
```python
def print_item(first, second, *args):
    print(first)
    print(second)
    print(args)

if __name__ == '__main__':
    print_item('first_item', 'second_item', 'third_item', 'forth_item', 'fifth_item')
```
**출력**
```
first_item
second_item
('third_item', 'forth_item', 'fifth_item')
```

*args에 해당하는 값이 tuple 형식 그대로 출력되는 것을 확인할 수 있습니다.

실제로 *args가 사용되는 대표적인 함수는 `print`입니다. *args 를 이용해서 가변인자를 받는 것을 확인할 수 있습니다.


**print 실제 code**
```python
def print(self, *args, sep=' ', end='\n', file=None): # known special case of print
    """
    print(value, ..., sep=' ', end='\n', file=sys.stdout, flush=False)

    Prints the values to a stream, or to sys.stdout by default.
    Optional keyword arguments:
    file:  a file-like object (stream); defaults to the current sys.stdout.
    sep:   string inserted between values, default a space.
    end:   string appended after the last value, default a newline.
    flush: whether to forcibly flush the stream.
    """
    pass
```


#### Keyword Argument

python은 인자를 keyword argument 형태로 받을 수도 있습니다. keyword argument를 unpacking 하기 위해서는 ** 가 사용됩니다.

****을 활용한 예제**
```python
def print_item(**kwargs):
    print(kwargs)
    print(kwargs.keys())
    print(kwargs.values())

if __name__ == '__main__':
    print_item(first=1, second=2)
```

**출력**
```
{'first': 1, 'second': 2}
dict_keys(['first', 'second'])
dict_values([1, 2])
```

keyword argument는 dictionary 형태로 값이 전달됩니다. 그래서 dictionary의 메서드인 `keys()`와 `values()`를 사용할 수 있습니다.


마지막으로 제일 많이 쓰이는 형태의 함수를 통해서 keyward argument와 unpacking이 어떻게 활용되는지 알아보도록 하겠습니다.

**예제 함수 - 1**
```python
def print_item(first, second, third):
    print(first)
    print(second)
    print(third)

if __name__ == '__main__':
    print_item(first='first_item', second='second_item', third='third_item')
```

**출력**
```
first_item
second_item
third_item
```

아래의 함수는 위의 함수와 동작이 동일합니다.
```python
def print_item(**kwargs):
    print(kwargs['first'])
    print(kwargs['second'])
    print(kwargs['third'])

if __name__ == '__main__':
    print_item(first='first_item', second='second_item', third='third_item')
```



**예제 함수 - 2**
```python
def print_item(**kwargs):
    print(kwargs)
    print(kwargs.keys())
    print(kwargs.values())

if __name__ == '__main__':
    print_item(first='first_item', second='second_item', third='third_item')
```

**출력**
```
{'first': 'first_item', 'second': 'second_item', 'third': 'third_item'}
dict_keys(['first', 'second', 'third'])
dict_values(['first_item', 'second_item', 'third_item'])
```

위의 함수는 또 아래와 동일합니다.

```python
def print_item(**kwargs):
    print(kwargs)
    print(kwargs.keys())
    print(kwargs.values())

if __name__ == '__main__':
    input_dict = {'first': 'first_item',
                  'second': 'second_item',
                  'third': 'third_item'}
    print_item(**input_dict)
```


가장 많이 쓰는 형태는 다음과 같습니다. keyword argument를 전달할 때, dictionary를 unpacking해서 전달하는 것입니다. 함수의 형태가 우리가 익숙한 모습이고 입력도 dictionary 형태여서 가독성이 좋아집니다.

**많이 사용되는 형태**
```python
def print_item(first, second, third):
    print(first)
    print(second)
    print(third)

if __name__ == '__main__':
    input_dict = {'first': 'first_item',
                  'second': 'second_item',
                  'third': 'third_item'}
    print_item(**input_dict)
```

