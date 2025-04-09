**과 .* 은 다르다   
> h*n	 | h 0번 이상 + n	         | "n", "hn", "hhhn"	"hamn",   | "hello"   
> h.*n |	h + (아무 문자)* + n	 | "hn", "hon", "hello_n"	      | "an", "hmm"   


*search 와 fullmatch는 다르다!*   
함수	설명
> re.search()	문자열 중 일부분이라도 매칭되면 True   
> re.fullmatch()	문자열 전체가 패턴과 일치해야 True   
> re.match()	문자열 시작부터 일치 (거의 안 씀)   

```python
import re

def main() :
    n = int(input())
    str_list = []
    pattern = input()
    pattern = pattern.replace("*",".*")
    for i in range(n) : 
        s = input()
        if re.fullmatch(pattern, s): print("DA")
        else : print("NE")

if __name__ == "__main__" : 
    main()

```
