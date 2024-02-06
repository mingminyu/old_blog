# Python 常用自定义装饰器

<show-structure depth="2" />

装饰器（Decorators）是 Python 中一种强大而灵活的功能，用于修改或增强函数或类的行为，它可以用于许多不同的情况，例如缓存、日志记录、权限控制等。通过在项目中使用的我们介绍的这些 Python 装饰器，可以简化我们的开发流程或者让我们的代码更加健壮。

装饰器本质上是一个函数，它接受另一个函数或类作为参数，并返回一个新的函数或类。它们通常用于在不修改原始代码的情况下添加额外的功能或功能。装饰器的语法使用 `@` 符号，将装饰器应用于目标函数或类，下面我们将介绍一些非常简单但是却很有用的自定义装饰器。

## 1. @timer: 测量执行时间

优化代码性能是非常重要的，`@timer` 装饰器可以帮助我们跟踪特定函数的执行时间。通过用这个装饰器包装函数，我可以快速识别瓶颈并优化代码的关键部分。将 `@timer` 与其他装饰器结合使用，可以全面地分析代码的性能。

```Python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"{func.__name__} took {end_time - start_time:.2f} seconds to execute.")
        return result
    return wrapper

@timer
def my_processing_func():
    ...
```


## 2. @memoize: 缓存结果

在数据科学中，我们经常使用计算成本很高的函数。`@memoize` 装饰器帮助我缓存函数结果，避免了相同输入的冗余计算，显著加快工作流程。

```Python
def memoize(func):
    cache = {}
 
    def wrapper(*args):
        if args in cache:
            return cache[args]

        result = func(*args)
        cache[args] = result
        return result
    return wrapper

@memoize
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```


## 3. @validate_input: 数据验证

数据完整性至关重要，`@validate_input` 装饰器可以验证函数参数，确保它们在继续计算之前符合特定的标准，可以方便的使用 `@validate_input` 在数据科学项目中一致地实现数据验证。

```Python
def validate_input(func):
    def wrapper(*args, **kwargs):
        # 数据验证主体逻辑
        if valid_data:
            return func(*args, **kwargs)
        else:
            raise ValueError("Invalid data. Please check your inputs.")
 
    return wrapper

@validate_input
def analyze_data(data):
    ...
```

## 4. @log_results: 日志输出

在运行复杂的数据分析时，跟踪每个函数的输出变得至关重要。`@log_results` 装饰器可以帮助我们记录函数的结果，以便于调试和监控。将 `@log_results` 与日志库结合使用，以获得更高级的日志功能。

```Python
def log_results(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        with open("results.log", "a") as log_file:
            log_file.write(f"{func.__name__} - Result: {result}\n")
        
        return result

    return wrapper

@log_results
def calculate_metrics(data):
    ...
```

## 5. @suppress_errors: 优雅的错误处理

数据科学项目经常会遇到意想不到的错误，可能会破坏整个计算流程。`@suppress_errors` 装饰器可以优雅地处理异常并继续执行。`@suppress_errors` 可以避免隐藏严重错误，还可以进行错误的详细输出，便于调试。

```Python
def suppress_errors(func):
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception as e:
            print(f"Error in {func.__name__}: {e}")
            return None
    return wrapper

@suppress_errors
def preprocess_data(data):
    ...
```

## 6. @validate_output: 确保质量结果

确保数据分析的质量至关重要，`@validate_output` 装饰器可以帮助我们验证函数的输出，确保它在进一步处理之前符合特定的标准，这样可以始终为验证函数输出定义明确的标准。

> 前面介绍的 `@validate_input` 是关注函数的参数输入值是否合规，只要合规的情况并不会关注该函数的输出结果是什么。
{style="note"}


```Python
def validate_output(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        
        if valid_output(result):
            return result
        else:
            raise ValueError("Invalid output. Please check your function logic.")
 
    return wrapper
 
@validate_output
def clean_data(data):
    ...
```


## 7. @retry: 重试执行

`@retry` 装饰器帮助我在遇到异常时重试函数执行，确保更大的弹性，使用 `@retry` 时应避免过多的重试。

```Python
import time
 
def retry(max_attempts: int, delay: int):
    def decorator(func):
        def wrapper(*args, **kwargs):
            attempts = 0
            while attempts < max_attempts:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Attempt {attempts + 1} failed. Retrying in {delay} seconds.")
                    attempts += 1
                    time.sleep(delay)
            raise Exception("Max retry attempts exceeded.")
        return wrapper
    return decorator

@retry(max_attempts=3, delay=2)
def fetch_data_from_api(api_url):
    ...
```


## 8. @visualize_results: 漂亮的可视化

`@visualize_results` 装饰器数据分析中自动生成漂亮的可视化结果。

```Python
import matplotlib.pyplot as plt
 
def visualize_results(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        plt.figure()
        # Your visualization code here
        plt.show()
        return result
    return wrapper


@visualize_results
def analyze_and_visualize(data):
    ...
```


## 9. @debug: 调试变得更容易

调试复杂的代码可能非常耗时，`@debug` 装饰器可以打印函数的输入参数和它们的值，以便于调试。

```Python
def debug(func):
    def wrapper(*args, **kwargs):
        print(f"Debugging {func.__name__} - args: {args}, kwargs: {kwargs}")
        return func(*args, **kwargs)
 
    return wrapper

@debug
def complex_processing(data, threshold=0.5):
    ...
```

## 10. @deprecated: 处理废弃的函数

随着我们的项目更新迭代，一些函数可能会过时，`@deprecated` 装饰器可以在一个函数不再被推荐时通知用户。

```Python
import warnings
 
def deprecated(func):
    def wrapper(*args, **kwargs):
        warnings.warn(f"{func.__name__} is deprecated and will be removed in future versions.", DeprecationWarning)
        return func(*args, **kwargs)
    return wrapper

@deprecated
def old_data_processing(data):
    ...
```

## 11. @admin_required: 访问控制

装饰器可以用来增加访问控制逻辑，例如验证用户权限。

```Python
def user_is_admin():
    # 这里应该是检查用户是否为管理员的逻辑
    return True

def admin_required(func):
    def wrapper(*args, **kwargs):
        if not user_is_admin():
            raise Exception("用户不是管理员，无法执行操作。")
        return func(*args, **kwargs)
    return wrapper

@admin_required
def sensitive_function():
    print("执行敏感操作。")

sensitive_function()
```


<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/dHHdEoYoCUAfs-uBxN8QEw">10个常用装器</a>
    <a href="https://mp.weixin.qq.com/s/Zr1XgLEKq1zVstkLGgfu7Q">5个高频 Python 装饰器使用场景</a>
</category>
<category ref="ref_github">
</category>
<category ref="ref_issues">
</category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>