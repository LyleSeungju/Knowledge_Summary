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
1 0 1 1 1 1
1 0 1 0 1 0
1 0 1 0 1 1
1 1 1 0 1 1
미로에서 1은 이동 할 수 있는 칸을 나타내고, 0은 이동할수 없는 칸
(1,1)에서 (N,M)의 위치로 이동 할 때 최소 칸 수를 구하시오

이동 할 수 있는 방법1
@ 0 @ @ @ 1
@ 0 @ 0 @ 0
@ 0 @ 0 @ 1
@ @ @ 0 @ @

이동 할 수 있는 방법2
@ 0 @ @ @ 1
@ 0 @ 0 @ 0
@ 0 @ 0 @ @
@ @ @ 0 1 @

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
로 구현 할 수 있다.
