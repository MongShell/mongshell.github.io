---
title:  "Python Decorator - 1"
toc: true
toc_label: "Table of Contents"
toc_stick: true

categories:
  - Python
tags:
  - Language
---

---
### Python Decorator

python 모듈을 개발 시, function name 을 남겨야 할 일이 있었습니다.
inspect library 를 사용해서 남길까도 했으나, 예쁘지도 않고, inspect의 용도와도 맞지 않는 것 같아 decorator로 만들어보았습니다.

decorator는 하나의 함수를 argument로 받아서 다른 함수를 반환하는 함수라고 할 수 있습니다.

테스트를 위한 코드는 다음과 같습니다. 

여기에서 `test()`와 `foo()`가 실행시마다 각 함수의 이름을 나타내는 것이 목표입니다.

**basic code**
```python
def foo():
    print('@@@@ foo @@@@')
    pass

def test():
    print('!!!! test !!!!')
    foo()

def main():
    test()

if __name__ == '__main__':
    main()
```

**출력**
```
!!!! test !!!!
@@@@ foo @@@@
```

decorator를 적용하는 방법은 아래와 같습니다.

**decorator 를 적용한 code**
```python
def logging_function_name(func):
    def echo_func():
        print(f'function name : {func.__name__}')
        return func()
    return echo_func

@logging_function_name
def foo():
    print('@@@@ foo @@@@')
    pass

@logging_function_name
def test():
    print('!!!! test !!!!')
    foo()

@logging_function_name
def main():
    test()

if __name__ == '__main__':
    main()
```

간단하게 살펴보면 `func`를 argument로 받아 `func`의 이름을 출력합니다.
그리고 적용하고 싶은 함수 위에 `@logging_function_name`만 추가해준다면 decorator 적용이 됩니다.

아래와 같이 함수가 실행되기 전에 함수의 이름을 출력하는 것을 확인할 수 있습니다.

**출력**
```
function name : main
function name : test
!!!! test !!!!
function name : foo
@@@@ foo @@@@
```

위에서 보다시피 decorator도 하나의 함수입니다. 그러면 decorator도 하나의 argument로 받을 수 있지 않을까요? decorator에 decorator를 적용하면 어떻게 될까요?

**decorator에 decorator를 적용한 code**
```python
@logging_function_name
@logging_function_name
def main():
    pass

if __name__ == '__main__':
    main()
```

아래처럼 decorator 자체 함수의 이름이 출력되는 것을 확인할 수 있습니다.

**출력**
```
function name : echo_func
function name : main

Process finished with exit code 0
```

원하는 결과를 얻었으니 우리가 만든 decorator가 어떻게 생겼는지 한 번 확인해보겠습니다.

**logging_function_name**
```python
def logging_function_name(func):
    def echo_func():
        print(f'function name : {func.__name__}')
        return func()
    return echo_func
```

`logging_function_name`은 함수를 argument로 받아서 그 함수를 return 하는 함수를 return 해줍니다. python 에서는 함수가 일급 객체라는 말을 들어보셨을텐데요. python 에서는 위와 같이 함수를 argument로 전달할 수 있습니다.
여기서는 `func`으로 함수를 받아서 `func.__name__` 을 이용해 함수의 이름을 출력하고 있습니다.

그런데 decorator를 검색해보면 *args, **kwargs 를 보신 경우가 있을 겁니다.
이것들은 함수의 argument를 받는 argument 입니다. 각각 positional argument 와 keyword argument를 나타내는데 이 내용은 `logging_function_name` 이 아닌 다른 decorator를 통해서 알아보도록 하겠습니다.

**logging_function_name - with argument**
```python
def logging_function_name(func):
    def echo_func(*args, **kwargs):
        print(f'function name : {func.__name__}')
        return func(*args, **kwargs)
    return echo_func
```