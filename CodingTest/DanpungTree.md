### [구름 Level 챌린지] 단풍 나무 

1. S(i, j) 값이 0이면 모든 나무가 물 들었음. 
2. S(i, j)의 단풍나무는 매일 아침마다 상하좌우에 모두 물들은 단풍나무 수 만큼 숫자가 줄어들고, 그 숫자가 S(i, j)값보다 크다면 해당 나무는 0이 된다.
3. 주어진 데이터에서 물들어 있는 나무가 한그루 이상일때 모든 나무가 물들려면 며칠 후 인지 구하시오.

#### 예제 설명
```js
 0 0 1
 2 2 0
 2 2 0
```
주어진 단풍나무가 위일 때
1. 첫째 날
	- S(1,3)의 주변엔 물든 나무가 2그루이지만 해당 위치의 나무는 1 그루뿐이므로 값은 0이 된다.
	- S(2,2)의 주변에 물든 나무가 2그루이고 해당 위치의 나무는 2 그루이므로 0이 된다.
	- S(2,1), S(3,2)의 주변의 물든 나무는 1그루인데, 해당 위치의 나무는 1그루이므로 1이 된다.
	- S(3,1)의 주변에 물든 나무는 없으므로 0 그대로
```js
 0 0 0
 1 0 0
 2 1 0
```
2. 둘째 날
```js
 0 0 0
 0 0 0
 2 0 0
```
3. 셋째 날
```js
0 0 0
0 0 0
0 0 0
```

3일째 되는날 모두 물들었으므로 모두 출력하게 된다.

### 입력
- 첫째 줄에는 크기를 나타내는 N
- 다음 N개의 줄에는 S(i, 1) ... S(i, N)이 주어진다.

우선 입력 데이터를 받아와 원하는 형태로 처리한다.
```js
const input = require("fs")
	.readFileSync("/dev/stdin")
	.toString()
	.trimEnd()
	.split("\n");

const n = Number(input[0]);
let data = [];
for (let i = 1; i <= n; i++) {
  data.push(input[i].split(" ").map(Number));
}
```

이후 주변 나무가 존재하는지 그리고 물든 나무인지 구별하는 함수
```js
function chk(a, b) {
	if (a >= n || b >= n || a < 0 || b < 0) {
		return 0;
	}
	if (data[a][b] == 0) {
		return 1;
	} else {
		return 0;
	}
}
```

이 함수가 실행될때마다 주어진 배열에 물든 나무가 있는지 없는 지 체크하고 물들도록 하였다.
또한 처리된 데이터가 기존 데이터에 영향을 주지 않게 복사하여 사용했다. 
```js
let resultDay = 0;
function day() {
	resultDay++;
	let newData = data.map(row => row.slice()); // data 배열의 복사본 생성
	for (let k = 0; k < n; k++) {
		for (let j = 0; j < n; j++) {
			if (data[k][j] != 0) {
				let count = chk(k + 1, j) + chk(k - 1, j) + chk(k, j - 1) + chk(k, j + 1);
				if (count > data[k][j]) {
					newData[k][j] = 0;
				} else {
					newData[k][j] -= count;
				}
			}
		}
	}
	data = newData; // data 배열을 업데이트
}
```


####실행
```js
while (true) {
	
	let isChk = false;
	for (let n1 = 0; n1 < n; n1++) {
		if (isChk) break;
		for (let n2 = 0; n2 < n; n2++) {
			if (data[n1][n2] != 0) {
				isChk = true;
				break;
			}
		}
	}
	
	if (!isChk) {
		console.log(resultDay);
		break;
	}
	day();
}
```

맨 처음엔 while이 실행되고 `day();` 함수를 먼저 실행하였다.
하지만 만약 주어진 데이터가 이미 모두 물든 나무였다면? 의도하지 않은 결과가 나오게 되었다. 따라서 나무를 물들게 하는 함수가 실행되기 전 해당 나무들이 모두 물들었는지 확인하고 물들지 않은 나무가 있다면 해당 프로세스가 실행되게끔 위치를 바꿨다. 

이 해결법은 O(n^3)의 시간 복잡도를 가졌기에 상당히 시간이 오래걸린다. 그러니 
다른 알고리즘을 이용하여 시간 복잡도를 줄이도록 생각해봐야겠다.
