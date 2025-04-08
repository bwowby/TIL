솔직히 절대 생각 못할 ...    
레이저가 나올때 지금까지 스택에 쌓인 막대 갯수만큼 잘린다 -> 막대기 하나가 끝날때는 해당 막대기 끝만 잘린다

``` python
def main() :
#()(((()())(())()))(())
        stick = list(input())
        print(stick)
        s = []

        keep = 0
        for i in range(len(stick)) : 
            if stick[i] == '(' : 
                s.append("(")

            else :  #)
                if stick[i-1] == '(' :  ##레이저
                    s.pop()
                    keep += len(s)
                else : #)) # 레이저가 아니면 막대기 끝난거니까 하나 댕강 해줌
                    keep += 1
                    s.pop()


        print(keep)

if __name__ == "__main__" : 
    main()
```
