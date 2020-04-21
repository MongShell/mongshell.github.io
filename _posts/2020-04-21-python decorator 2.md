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

전 포스트에서는 positional argument와 keyword argument가 없는 decorator로 함수의 이름을 출력하는 decorator를 만들었습니다.
이번에는 argument를 조작해서 decorator가 할 수 있는 일을 알아보도록 하겠습니다.

그 전에 positional argument와 keyword argument에 대해 간단한 설명이 필요합니다.

positional argument는 함수의 위치에 따른 argument를 의미합니다.
`add(1,2)`에서 첫번째 argument가 1, 두번째 argument가 2 라고 할 때 이 argument들을 나타냅니다.

keyword argument는 keyword 로 전달된 argument를 의미합니다.
`add(i=1, j=2)`에서 `i=1, j=2`, 즉, i 에 대입되는 argument가 1, j 에 대입되는 argument가 2라는 것을 의미합니다.

아래의 decorator로 간단히 확인해보겠습니다.

간단한 `add` 함수와 decorator를 만들었습니다. 현재 decorator는 아무 역할을 하지 않습니다.

**basic decorator**
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

**출력**
```
decorator
add function
result : 3
```

여기서 args와  kwargs를 출력하면 어떻게 나올까요?
decorator를 아래와 같이 수정해서 args 와 kwargs 에 대해서 파악해보겠습니다.

**argumnet 의 값, type 출력**
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

**출력**
```
decorator
type(args) : <class 'tuple'>
args : (1, 2)
type(kwargs) : <class 'dict'>
kwargs : {}
add function
result : 3
```

위와 같이 args는 `tuple`타입, kwargs 는 `dict`타입이라는 것을 알 수 있습니다.

여기서 kwargs가 `{}`인 이유는, argument가 positional argument로 전달됐기 때문입니다. 
`add(i=1, j=2)`로 실행했다면 args가 `()`, kwargs가 `kwargs : {'i': 1, 'j': 2}` 로 출력되는 것을 확인할 수 있습니다.

decorator는 이렇게 argument로 넘어온 값들을 이용해 연산이 가능합니다. 물론, 그 연산의 결과도 전달할 수 있습니다. 
이번에는 decorator에 args 값을 인위적으로 변경해보도록 하겠습니다.

**positional argument 에 임의의 값 대입**
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

**출력**
```
decorator
add function
result : 300
```

결과가 300 이 나온 것을 확인할 수 있습니다.

decorator에 kwargs값을 인위적으로 변경해보도록 하겠습니다.

**keyword argument 에 임의의 값 대입**
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

**출력**
```
decorator
add function
result : 3000
```

결과가 3000 이 나온 것을 확인할 수 있습니다.

---------------

#### 참고

##### argument 형태
positional argument와 keyword argument의 값이 꼭 *args, **kwargs 일 필요는 없습니다.
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
위와 같이 변수명은 동작에 영향을 끼치지 않습니다. 하지만 다만, `*`의 개수는 positional argument가 1개, keyword argument가 2개로 정해져있습니다. 이와 관련된 내용은 다른 포스트에서 다루도록 하겠습니다.

##### positional argument와 keyword argument

positional argument와 keyword argument도 다른 포스트에서 설명할 예정입니다만,
위의 decorator를 이해할 수 있는 정도로 간단히 설명하겠습니다.

위에서 활용했던 `add` 함수를 positional argument 를 이용해서 출력하는 예제입니다.

**positional argument 활용**
```python
def add(i, j):
    print('add function')
    return i + j

if __name__ == '__main__':
    tuple_input = (1, 2)
    print(f'tuple_input : {tuple_input}')
    print(f'result : {add(*tuple_input)}')
```

**출력**
```
tuple_input : (1, 2)
*tuple_input :  1 2
add function
result : 3
```

이때 tuple을 그냥 argument로 넘기는 것이 아니라 `add(*tuple_input)`처럼 `*` 하나를 tuple 앞에 붙이는 것을 볼 수 있습니다.
이것을 unpacking이라고 하는데, tuple을 unpacking 해서 argument로 넘길 때는 `*`가 하나 필요합니다.

이번에는 `add` 함수를 keyword argument 를 이용해서 출력하는 예제입니다.

**keyword argument 활용**
```python
def add(i, j):
    print('add function')
    return i + j

if __name__ == '__main__':
    dict_input = {"i": 100, "j": 200}
    print(f'dict_input : {dict_input}')
    print(f'result : {add(**dict_input)}')
```

**출력**
```
dict_input : {'i': 100, 'j': 200}
add function
result : 300
```

dictionary를 argument로 넘길 때는 `add(**dict_input)`처럼 `*` 두개를 dictionary 앞에 붙이는 것을 볼 수 있습니다.
dictionary를 unpacking 해서 argument로 넘길때는 `*`가 두개 필요합니다.

이 때 dictionary의 key가 함수 parameter의 keyword와 일치해야 합니다.
다시 말하면, `add(i, j)` 인 경우 dictionary의 key값이 `i`와 `j`여야 한다는 의미입니다.