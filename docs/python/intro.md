# Biến và kiểu dữ liệu trong Python

## Khai báo biến

Python không cần khai báo kiểu dữ liệu:

```python
# Số nguyên
age = 25

# Số thực
price = 19.99

# Chuỗi
name = "John"

# Boolean
is_student = True
```

## Kiểm tra kiểu dữ liệu

```python
print(type(age))         # <class 'int'>
print(type(price))       # <class 'float'>
print(type(name))        # <class 'str'>
print(type(is_student))  # <class 'bool'>
```

## Chuyển đổi kiểu dữ liệu

```python
# String sang int
num_str = "100"
num_int = int(num_str)

# Int sang string
age = 25
age_str = str(age)

# Int sang float
x = 10
x_float = float(x)
```
