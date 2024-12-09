---
title: "Python并行计算"
categories:
  - Blog
  - Tech
tags:
  - Python
  - Parallel computing
---

# Python并行计算示例代码(带进度条)

```python
import concurrent.futures
from tqdm.auto import tqdm

def add(a: int):
    return a + a
    
def parallel_add():

    a = [i for i in range(1000000)]

    with concurrent.futures.ProcessPoolExecutor(max_workers=24) as executor:
        result = list(tqdm(executor.map(add, a), total=a.__len__()))

if __name__ == '__main__':
    parallel_add()
```