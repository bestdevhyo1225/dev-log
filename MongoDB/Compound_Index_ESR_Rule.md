# Compound Index, E-S-R Rule

## E-S-R Rule

- E: Equality(동등)
- S: Sort(정렬)
- R: Range(범위)

### E (Equality First)

```javascript
// 인덱스 생성
db.games.createIndex({ gamertag: 1 });

// find 쿼리
db.games.find({ gamertag: "Ace" })
```

### E -> R (Equality Before Range)

```javascript
// 인덱스 생성
db.games.createIndex({ gamertag: 1, level: 1 });

// find 쿼리
db.games.find({ gamertag: "Ace", level: { $gt: 50 } })
```

### E -> S (Equality Before Sort)

```javascript
// 인덱스 생성
db.games.createIndex({ gamertag: 1, score: 1 });

// find 쿼리
db.games.find({ gamertag: "Ace" }).sort({ score: 1 })
```

### S -> R (Sort Before Range)

- 왜 Sort를 먼저하는 것이 좋은지 의문이 들지만, `R -> S` 에 의한 Sort 작업이 `S -> R` 에 의한 Range 작업보다 더 큰 비용이라고 판단하는 경우가 있다.
- 즉, 마지막 Sort 작업보다 Index Key를 더 많이 읽는 것이 효율적이라고 판단하기 때문에 `S -> R` 규칙을 따라야한다.

```javascript
// 인덱스 생성 (S -> R)
db.games.createIndex({ score: 1, level: 1 });

// find 쿼리
db.games.find({ level: { $gt: 50 } }).sort({ score: 1 })
```

### E -> S -> R (Equality Sort Range)

```javascript
// 인덱스 생성 (E -> S -> R)
db.games.createIndex({ gamertag: 1, score: 1, level: 1 });

// find 쿼리
db.games.find({ gamertag: "Ace", level: { $gt: 50 } }).sort({ score: 1 })
```

## E-S-R Rule이 항상 성립할까?

- 아래의 예시에 따르면 `date` 필드 값이 1개라면, 1개를 조회하기 위한 `score` 필드를 모두 조회하는 불 필요한 탐색이 필요하다.

```javascript
// 인덱스 생성 (E -> S -> R)
db.games.createIndex({ gamertag: 1, score: 1, date: 1 });

// find 쿼리
db.games.find({ gamertag: "Ace", date: { $gte: 2022 } }).sort({ score: 1 })
```

- 이에 따라 아래의 인덱스 조건으로 생성하면, 1개의 Index Key만 탐색하게 된다. 

```javascript
// 인덱스 생성 (E -> R)
db.games.createIndex({ gamertag: 1, date: 1 });

// find 쿼리
db.games.find({ gamertag: "Ace", date: { $gte: 2022 } }).sort({ score: 1 })
```

- 즉, `E-S-R Rule` 이 항상 성립하는 것은 아니고, 일반적인 상황에서 유리하게 사용할 수 있다는 점을 기억해야한다.
