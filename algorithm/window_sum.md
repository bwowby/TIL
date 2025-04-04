```python
def main() :
    n,m = map(int,input().split())
    arr = list(map(int,input().split()))

    window_sum = sum(arr[:m])
    max_sum = window_sum

    for i in range(m,n) : 
        window_sum = window_sum - arr[i-m] + arr[i]
        if max_sum < window_sum : 
            max_sum = window_sum

    print(max_sum)

if __name__ == "__main__" : 
    main()
```

for 두번 거치지 않고 윈도우로 한칸씩 이동하면서 앞뒤로 더하고 빼는 로직!
