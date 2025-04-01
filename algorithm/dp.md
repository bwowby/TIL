## dp 처음부터 배열 만들지 않아도됨!

```python

def main() :
    n = int(input())
    
    if n < 3 : 
        print(1)
    else : 
        array = [1,1,1]

        for i in range(3,n) :
            array.append( array[i-1] + array[i-3] ) # 그 수만큼 append 해주기!
        print(array[n-1])   

if __name__ == "__main__" : 
    main()   
```
