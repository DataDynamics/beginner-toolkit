# Python Language

## main()

Java의 main 메소드 처럼 Python에서도 다음과 같이 main 함수를 추가할 수 있습니다.

```python
def main():
    print("Hello from main!")

if __name__ == "__main__":
    main()
```

이제 다음과 같이 실행할 수 있습니다.

```shell
$ python myscript.py
# 출력: Hello from main!
```

main 함수를 지정하지 않더라도 실행은 가능하다 import를 하게 되면 코드가 실행이 될 수 있기 때문에 재사용을 목적으로 구조화 시킨다면 main 함수를 통해서 실행하는 것이 좀더 바람직합니다.

## self, cls

`self`, `cls`는 자기 자신을 참고하는 용도로 사용

* `self`
  * `self`는 클래스의 인스턴스(객체) 자신을 가리킵니다.
  * `self`는 인스턴스 메서드의 첫 번째 인자로 자동으로 전달됩니다.
* `cls`
  * `cls`는 클래스 메서드에서 클래스 그 자체를 참조합니다.
  * `@classmethod`로 데코레이션된 메서드의 첫 번째 인자는 자동으로 `cls`입니다.

## self

```python
class Car:
    def __init__(self, brand):
        self.brand = brand  # 인스턴스 변수에 접근

    def drive(self):
        print(f"{self.brand} is driving.")


car1 = Car("Hyundai")
car1.drive()  # Hyundai is driving.
```

여기서 `self.brand`는 `car1.brand`와 같습니다. 즉, 메서드 내부에서 `self`는 `car1`을 참조합니다.


## cls

```python
class Dog:
    species = "Canine"

    @classmethod
    def create_dog(cls, name):
        print(f"Creating a new {cls.species} named {name}")
        return cls()  # 새로운 인스턴스를 생성 가능


Dog.create_dog("Buddy")  # Creating a new Canine named Buddy
```

`cls`는 Dog 클래스를 참조합니다. 이를 통해 클래스 변수에 접근하거나, 클래스의 생성자를 호출할 수 있습니다.

## 모듈화

프로젝트의 규모가 커지거나, 역할에 따라서 기능이 잘 분리되어 있을때 재사용을 위해서 모듈화 형태로 설계할 수 있습니다.
Python 코드를 모듈화하고 import로 불러와 재사용하려면, 디렉토리 구조를 설계하고 __init__.py, 모듈 파일 분리, 패키지화 등의 과정을 따라야 합니다. 아래는 Python 프로젝트를 모듈화하고 import 가능하게 구조화하는 방법에 대해서 알아봅니다.

### 기본 용어

* 모듈(Module): `.py` 파일 하나가 하나의 모듈입니다.
* 패키지(Package): `__init__.py`가 있는 디렉토리 → 여러 모듈을 포함할 수 있음.
* `import`: 다른 파일에 정의된 함수, 클래스, 변수 등을 가져오기 위한 문법.

### 디렉토리 구조

```
myproject/
│
├── main.py                    ← 실행 파일
├── requirements.txt
├── utils/
│   ├── __init__.py            ← 패키지로 인식시키는 파일
│   ├── file_utils.py          ← 모듈
│   └── math_utils.py          ← 모듈
├── models/
│   ├── __init__.py
│   └── linear_model.py
```

### 모듈의 정의

`utils/file_utils.py` 파일을 다음과 같이 생성할 수 있습니다.

```python
def read_file(path):
    with open(path, 'r') as f:
        return f.read()
```

`utils/math_utils.py` 파일을 다음과 같이 생성할 수 있습니다.

```python
def add(a, b):
    return a + b
```

### Import

다음은 개별 모듈의 함수를 import할 수 있습니다.

```python
from utils.file_utils import read_file
from utils.math_utils import add

print(read_file("data.txt"))
print(add(10, 5))
```

다음과 같이 디렉토리를 통으로 import할 수 있습니다.

```python
import utils.math_utils as mu
print(mu.add(2, 3))
```

### __init__.py의 역할

* Python 3.3 이상에서는 필수는 아니지만, 하위 호환성과 명시적인 패키지 인식을 위해 `__init__.py`는 넣는 것이 좋습니다.
* `__init__.py` 내부에서 모듈을 미리 `import` 해두면 편하게 쓸 수 있습니다:

`utils/__init__.py` 파일에 다음을 추가합니다.

```python
from .file_utils import read_file
from .math_utils import add
```

그러면 다음과 같이 사용할 수 있습니다.

```python
from utils import read_file, add
```

## PYTHONPATH

* `PYTHONPATH`는 Python 인터프리터가 모듈을 찾을 때 참조하는 경로 목록에 영향을 주는 환경 변수입니다. 쉽게 말해, Python이 import 시에 어디서 모듈을 찾아야 하는지를 지정하는 방법입니다.
* 기본적으로 Python은 표준 라이브러리나 `site-packages`와 같은 기본 경로에서 모듈을 찾습니다.
* 그러나 사용자 정의 모듈이나 외부 디렉토리에 있는 모듈을 불러오고 싶을 때, 해당 디렉토리를 `PYTHONPATH`에 추가하면 됩니다.

### PYTHONPATH 동작 순서

1.	현재 실행 중인 스크립트의 디렉토리
2.	`PYTHONPATH`에 설정된 경로 (환경 변수)
3.	Python의 표준 라이브러리 경로
4.	`site-packages` 경로 등

경로 목록을 확인하려면 다음의 코드를 실행할 수 있습니다.

```python
import sys
print(sys.path)
```

### PYTHONPATH 설정 방법

```
export PYTHONPATH=/your/custom/path
python script.py
```
다음과 같이 동적으로 스크립트 디렉토리를 추가할 수 있습니다.

```
# main.py
import sys
sys.path.append("./mylib")

from mymodule import some_function
```







