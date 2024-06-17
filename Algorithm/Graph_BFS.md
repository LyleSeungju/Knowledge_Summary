그래프는 정점과 간선으로 이뤄진 자료구조의 일종이다. 
- 정점: 위치라는 개념
- 간선: 위치 간의 관계 / 정점과 정점을 연결한 선

1. 무방향 그래프
2. 방향 그래프 

트리와 그래프의 차이점: 여러개의 정점을 맺을 수 있는 그래프와 달리 트리는 각 노드가 하나의 부모 노드만을 갖는다. 
	개념 - 트리 < 그래프

#### BFS 너비 우선 탐색 Breadth First Search
: 현재 정점에 연결된 가까운 점부터 탐색
- 그래프 모형은 배열 사용 / BFS 알고리즘은 Queue 사용
	- JS에서 Queue를 사용하는 방법
		1. Array.shift() 사용
		2. Object 사용하여 직접 구현
			```js
			class Queue {
				constructor(){
					this.storage = new Object();
					this.front = 0;
					this.rear = 0;
				}
				size(){
					return this.rear - this.front;
				}
				enqueue(element){
					this.storage[this.rear] = element;
					this.rear++;
				}
				dequeue(){
					let data = this.storage[this.front];
					delete this.storage[this.front];
					this.front++;
					if(this.front == this.rear){
						this.front = 0;
						this.rear = 0;
					}
					return data;
				}
			}
			
			```
- 한번 방문한 정점은 다시 택하지 않는다.

탐색과정 ![[Pasted image 20240617195142.png]]
1.  1을 큐에 넣는다 `Queue[1]`
2. 1에 연결된 가장 가까운 점 2,3,4를 큐에 추가. > 1 삭제 `1out Queue[2,3,4]`
3. 2에 연결된 가장 가까운 점 5,6를 큐에 추가. > 2 삭제 `2out Queue[3,4,5,6]`
4. 3에 연결된 가장 가까운 점이 없다. > 3삭제 `3out Queue[4,5,6]`
5. 4에 연결된 가장 가까운 점 7,8를 큐에 추가. > 4 삭제 `4out Queue[5,6,7,8]`
6. 5에 연결된 가장 가까운 점 9,10를 큐에 추가 > 5삭제 `5out Queue[6,7,8,9,10]`
7. 6에 연결된 가장 가까운 점이 없다. > 6삭제 `6out Queue[7,8,9,10]`
8. 7에 연결된 가장 가까운 점 11,12 큐에 추가 > 7 삭제 `7out Queue[8,9,10,11,12]`
9. 8과 연결된 가장 가까운 점이 없다. > 8 삭제 `8out Queue[9,10,11,12]`
10. 9과 연결된 가장 가까운 점이 없다. > 9 삭제 `9out Queue[10,11,12]`
11. 10과 연결된 가장 가까운 점이 없다. > 10 삭제 `10out Queue[11,12]`
12. 11과 연결된 가장 가까운 점이 없다. > 11 삭제 `11out Queue[12]`
13. 12과 연결된 가장 가까운 점이 없다. > 12 삭제 `12out Queue[]`

 정점 1에 거리가 0이라면
 - 2,3,4 거리는 1
 - 5,6,7,8 거리는 2
 - 9,10,11,12 거리는 3

시간 복잡도는 한 정점에 연결된 모든 정점을 탐색 > O(이동 가능한 정점의 개수)

##### 미로 탐색
NxM 크기의 배열로 표현되는 미로
```
1 0 1 1 1 1
1 0 1 0 1 0
1 0 1 0 1 1
1 1 1 0 1 1
```
미로에서 1은 이동 할 수 있는 칸을 나타내고, 0은 이동할수 없는 칸
(1,1)에서 (N,M)의 위치로 이동 할 때 최소 칸 수를 구하시오

이동 할 수 있는 방법1
```
@ 0 @ @ @ 1
@ 0 @ 0 @ 0
@ 0 @ 0 @ 1
@ @ @ 0 @ @
```
이동 할 수 있는 방법2
```
@ 0 @ @ @ 1
@ 0 @ 0 @ 0
@ 0 @ 0 @ @
@ @ @ 0 1 @
```
이것을 자바스크립트로 구현한다면

```js
class Queue {
    constructor() {
        this.storage = {};
        this.front = 0;
        this.rear = 0;
    }
    
    size() {
        return this.rear - this.front;
    }
    
    enqueue(element) {
        this.storage[this.rear] = element;
        this.rear++;
    }
    
    dequeue() {
        let data = this.storage[this.front];
        delete this.storage[this.front];
        this.front++;
        if (this.front === this.rear) {
            this.front = 0;
            this.rear = 0;
        }
        return data;
    }
}

function bfs(maze, N, M) {
    let directions = [ [-1, 0], [1, 0], [0, -1], [0, 1] ];
    let queue = new Queue();
    queue.enqueue([0, 0]);
    
    while (queue.size() > 0) {
        let [x, y] = queue.dequeue();
        
        for (let k = 0; k < directions.length; k++) {
            let dx = x + directions[k][0];
            let dy = y + directions[k][1];
            
            if (dx >= 0 && dx < N && dy >= 0 && dy < M && maze[dx][dy] === 1) {
                maze[dx][dy] = maze[x][y] + 1;
                queue.enqueue([dx, dy]);
            }
        }
    }
    
    return maze[N-1][M-1];
}

let maze = [
    [1, 0, 1, 1, 1, 1],
    [1, 0, 1, 0, 1, 0],
    [1, 0, 1, 0, 1, 1],
    [1, 1, 1, 0, 1, 1]
];

let N = 4;
let M = 6;

console.log(bfs(maze, N, M)); 
```
로 구현 할 수 있다. - 출처: 보통의 취준생을 위한 코딩테스트 with 파이썬

이후 나는 해당 코드의 동작에 대해 이해하려했다.

1. 첫번째 반복일때
 - 현재 위치(큐) : 0, 0
 - 큐에서 현재 위치를 꺼내어 네 방향으로 탐색
   - 위 -1, 0 : 범위 밖
   - 아래 1, 0 : 유효 maze[1][0]을 2로 업데이트하고 1,0을 큐에 추가
   - 왼쪽 0, -1 : 범위 밖
   - 오른쪽 0, 1 : 벽
  
2. 두번째 반복
- 현재 위치(큐) : 1, 0
- 큐에서 현재 위치를 꺼내 네 방향 탐색
  - 위 0, 0 : maze[0][0]의 값은 1
  - 아래 2, 0 : maze[2][0] 값은 1
  - 왼쪽 1, -1 : 범위 밖
  - 오른쪽 1, 1 : 벽
  
하지만 이 코드에서 나는 두번째 반복일때 maze[0][0]이 값이 1이라면 다시 이전 노드로 진행될 수도 있지 않을까? 라는 생각이 들었고,
이전 노드를 이미 방문했다는 확실한 구별방법이 있어야겠다는 생각이 들었다.
```js
function bfs(maze, N, M) {
    let directions = [ [-1, 0], [1, 0], [0, -1], [0, 1] ];
    let queue = new Queue();
    queue.enqueue([0, 0]);
    let visited = Array.from(Array(N), () => Array(M).fill(false));
    visited[0][0] = true;
    maze[0][0] = 1;

    while (queue.size() > 0) {
        let [x, y] = queue.dequeue();
        
        for (let k = 0; k < directions.length; k++) {
            let dx = x + directions[k][0];
            let dy = y + directions[k][1];
            
            if (dx >= 0 && dx < N && dy >= 0 && dy < M && !visited[dx][dy] && maze[dx][dy] === 1) {
                visited[dx][dy] = true;
                maze[dx][dy] = maze[x][y] + 1;
                queue.enqueue([dx, dy]);
            }
        }
    }
    
    return maze[N-1][M-1];
}
```
이 코드는 N x M 형태의 visited 배열을 만들어 이미 한번 방문한 노드라면 true값으로 설정하여 다시 방문하지 않게 하였다.
따라서
```js
let maze = [
    [1, 0, 1, 1, 1, 1],
    [1, 0, 1, 0, 1, 0],
    [1, 0, 1, 0, 1, 1],
    [1, 1, 1, 0, 1, 1]
];

let N = 4;
let M = 6;
```
로 한다면 

```js
maze: // 미로
1 0 9 10 11 12
2 0 8 0 12 0
3 0 7 0 13 14
4 5 6 0 14 15

visited: // 방문한 배열
T F T T T T
T F T F T F
T F T F T T
T T T F T T
```
로 되어 최종적으로 N,M까지의 최단 거리는 15가 된다. 
