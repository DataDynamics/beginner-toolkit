# Python Language

## self, cls

`self`, `cls`는 자기 자신을 참고하는 용도로 사용

* `self`
  * self는 클래스의 인스턴스(객체) 자신을 가리킵니다.
	•	self는 인스턴스 메서드의 첫 번째 인자로 자동으로 전달됩니다.
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

