

# 排序

## 快速排序

```py
def quicksort(arr):
    if len(arr) < 1:
        return arr
    pivot = arr[len(arr) // 2]  # python3 中不能使用 /, python2 可以
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    return quicksort(left) + middle + quicksort(right)


if __name__ == '__main__':
    print(quicksort([3, 6, 8, 10, 1, 2, 1]))  # [1, 1, 2, 3, 6, 8, 10]
```
