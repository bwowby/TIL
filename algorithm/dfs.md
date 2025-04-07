한차례 돌때 마다 가장 바깥에서 확인할 수 있는 부분이 필요함으로 가장 밖에서 시작하고 visit 확인하면서 dfs 호출하기

```python
def main() :
    w = 1 
    h = 1

    def dfs(i,j) :
        visit[i][j] = 1
        for ii in [-1,-1,-1,0,0,1,1,1] :
            for jj in [-1,0,1,-1,1,-1,0,1] :
                if i+ii >= 0 and i+ii < h and j+jj >= 0 and j+jj < w :
                    
                    if arr[i+ii][j+jj] == 1 and visit[i+ii][j+jj] == 0 :
                        dfs(i+ii,j+jj)
                    else : continue

    while (w!=0 and h!=0 ):
        w,h = map(int,input().split())
        print(w,h)
        if w==0 and h==0 : 
            return
        else : 
            island = 0
            arr = []
            visit = []

            for i in range(h) :
                arr.append(list(map(int,input().split())))
                visit.append([0]*w)

            # 여기서 돌면서 한 차례씩 끝난걸 체크해줘야함 
            for y in range(h) :   
                for x in range(w) : 
                    if visit[y][x] == 0 and arr[y][x] == 1 :
                        dfs(y,x)
                        island += 1
                        print(visit)
                    else : continue
        print(island)

if __name__ == "__main__" : 
    main()


```
