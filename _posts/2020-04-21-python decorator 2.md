---
title:  "Python Decorator - 2"
toc: true
# toc_label: "Table of Contents"
toc_stick: true

categories:
  - Python
tags:
  - Language
---

### Python Decorator

전 포스트에서는 `positional argument`와 `keyword argument`가 없는 `decorator`로 함수의 이름을 출력하는 decorator 를 만들었는데 이번에는 argument 를 조작해서 decorator 가 할 수 있는 일을 알아보도록 하겠습니다.

그 전에 `positional argument` 와 `keyword argument` 에 대해 간단한 설명이 필요할 것 같습니다.

`positional argument` 는 함수의 위치에 따른 인자를 나타냅니다.
`add(1,2)` 에서 첫번째 인자가 1, 두번째 인자가 2 라고 할 때 이 것을 나타냅니다.

`keyword argument` 는 keyword 에 어떤 인자가 들어왔느냐를 dictionary 형태로 받습니다.
`add(i=1, j=2)` 에서 i 에 대입되는 인자가 1, j 에 대입되는 인자가 2 라는 것을 나타냅니다.

아래의 decorator 로 간단히 확인해보겠습니다.

간단한 add 함수와 decorator 를 만들었습니다. 현재 decorator 는 아무 역할을 하지 않습니다.

```python
def decorator(func):
    def echo_func(*args, **kwargs):
        print('decorator')
        return func(*args, **kwargs)
    return echo_func

@decorator
def add(i, j):
    print('add function')
    return i + j

if __name__ == '__main__':
    print(f'result : {add(1, 2)}')
```

출력
```
decorator
add function
result : 3
```


여기서 `args` 와  `kwargs` 를 출력하면 어떻게 나올까요?

decorator 를 아래와 같이 수정해서 args 와 kwargs 에 대해서 파악해보도록 하겠습니다.


```python
def decorator(func):
    def echo_func(*args, **kwargs):
        print('decorator')
        print(f'type(args) : {type(args)}')
        print(f'args : {args}')
        print(f'type(kwargs) : {type(kwargs)}')
        print(f'kwargs : {kwargs}')
        return func(*args, **kwargs)
    return echo_func
```

출력
```
decorator
type(args) : <class 'tuple'>
args : (1, 2)
type(kwargs) : <class 'dict'>
kwargs : {}
add function
result : 3
```

위와 같이 args 는 tuple 타입, kwargs 는 dict 타입이라는 것을 알 수 있습니다. 그리고 이 값을 이용해서 decorator 가 원하는 작업을 할 수도 있습니다. 여기서 kwargs 가 `{}` 로만 나온 이유는, argument 가 position 으로 전달됐기 때문입니다. `add(i=1, j=2)` 로 실행했다면 args 가 `()`, kwargs 가 `kwargs : {'i': 1, 'j': 2}` 로 출력되는 것을 보실 수 있습니다.


그러면 저 값을 변경하는 것으로 add 함수의 출력에 영향을 줄 수 있을까요? 네. 가능합니다.

decorator 에 args 값을 인위적으로 변경해보도록 하겠습니다.

```python
def decorator(func):
    def echo_func(*args, **kwargs):
        print('decorator')
        args = (100, 200)
        return func(*args, **kwargs)
    return echo_func

@decorator
def add(i, j):
    print('add function')
    return i + j

if __name__ == '__main__':
    print(f'result : {add(1, 2)}')
```

출력
```
decorator
add function
result : 300
```

결과가 300 이 나온 것을 확인할 수 있습니다.

decorator 에 kwargs 값을 인위적으로 변경해보도록 하겠습니다.

```python
def decorator(func):
    def echo_func(*args, **kwargs):
        print('decorator')
        kwargs['i'] = 1000
        kwargs['j'] = 2000
        return func(*args, **kwargs)
    return echo_func

@decorator
def add(i, j):
    print('add function')
    return i + j

if __name__ == '__main__':
    print(f'result : {add(i=1, j=2)}')
```

출력
```
decorator
add function
result : 3000
```

결과가 3000 이 나온 것을 확인할 수 있습니다.


---------------
`positional argument` 와 `keyword argument` 의 값이 꼭 `*args`, `**kwargs` 일 필요는 없습니다.
예를 들어
```python
def decorator(func):
    def echo_func(*abce, **defg):
        print('decorator')
        defg['i'] = 1000
        defg['j'] = 2000
        return func(*abce, **defg)
    return echo_func
```
이런 형태여도 동작에는 전혀 문제없습니다. 다만, 위와 같이 쓰면 다른 개발자들이 알아보기 어려울겁니다.

---------------
`positional argument` 와 `keyword argument` 에 관해 짤막하게 설명을 더해볼까 합니다.

사실 `positional argument` 와 `keyword argument` 만으로도 길게 설명해야 하지만, 여기서는 위의 decorator 를 이해할 수 있는 정도로만 설명하겠습니다.

위에서 활용했던 `add` 함수를 `positional argument` 를 이용해서 출력하는 예제입니다.

```python
def add(i, j):
    print('add function')
    return i + j

if __name__ == '__main__':
    tuple_input = (1, 2)
    print(f'tuple_input : {tuple_input}')
    print(f'result : {add(*tuple_input)}')
```

출력
```
tuple_input : (1, 2)
*tuple_input :  1 2
add function
result : 3
```

위와 같이 asterisk 를 이용해서 tuple 을 넘겨서 함수에서 연산을 할 수 있습니다.

마찬가지로 `add` 함수를 `keyword argument` 를 이용해서 출력하는 예제입니다.

```python
def add(i, j):
    print('add function')
    return i + j

if __name__ == '__main__':
    dict_input = {"i": 100, "j": 200}
    print(f'dict_input : {dict_input}')
    print(f'result : {add(**dict_input)}')
```

출력
```
dict_input : {'i': 100, 'j': 200}
add function
result : 300
```

이 때 dictionary 의 key 가 함수의 parameter 의 keyword 와 일치해야 합니다.
다시 말하면, `add(i, j)` 인 경우 dictionary 의 key 값이 `i` 와 `j` 여야 한다는 의미입니다.