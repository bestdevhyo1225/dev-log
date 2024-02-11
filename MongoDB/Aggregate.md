# Aggreate

## 예제

```json
[
  {
    "_id": 500,
    "scores": [
      {
        "type": "quiz",
        "avg_score": 51.35603805996617
      },
      {
        "type": "exam",
        "avg_score": 45.04082924638036
      }
    ]
  },
  {
    "_id": 499,
    "scores": [
      {
        "type": "exam",
        "avg_score": 52.234488613263466
      },
      {
        "type": "quiz",
        "avg_score": 48.96268506750967
      }
    ]
  },
  {
    "_id": 498,
    "scores": [
      {
        "type": "quiz",
        "avg_score": 53.827492248151465
      },
      {
        "type": "exam",
        "avg_score": 48.51775335555769
      }
    ]
  },
  {
    "_id": 497,
    "scores": [
      {
        "type": "quiz",
        "avg_score": 51.27682967858154
      },
      {
        "type": "exam",
        "avg_score": 50.80561533355925
      }
    ]
  },
  {
    "_id": 496,
    "scores": [
      {
        "type": "exam",
        "avg_score": 47.28546854417578
      },
      {
        "type": "quiz",
        "avg_score": 50.30975687853305
      }
    ]
  }
]
```

위의 결과가 나오도록 쿼리를 작성하세요

### 쿼리1

```javascript
db.grades.aggregate([
    {
        $unwind: "$scores" // array -> object 타입으로 변환해준다.
    },
    {
        $match: {
            "scores.type": {
                $in: ["exam", "quiz"]
            }
        }
    },
    {
        $group: {
            _id: {
                class_id: "$class_id",
                type: "$scores.type"
            },
            avg_score: {
                $avg: "$scores.score"
            }
        }
    },
    {
        $group: {
            _id: "$_id.class_id",
            scores: {
                $push: {
                    type: "$_id.type",
                    avg_score: "$avg_score"
                }
            }
        }
    },
    {
        $sort: {
            _id: -1
        }
    },
    {
        $limit: 5
    }
]);
```

### 쿼리2

```javascript
db.grades.aggregate([
    {
        $addFields: {
            tmp_scores: { // scores 필드 값을 tmp_scores 필드 값에 넣는다.
                $filter: {
                    input: "$scores", // scores 필드 값을
                    as: "scores_var", // scores_var 변수에 넣는다.
                    cond: { // 아래 조건에 해당하는 scores 필드 값만 tmp_scores 필드 값에 넣을 수 있다.
                        $or: [
                            { $eq: ["$$scores_var.type", "exam"] },
                            { $eq: ["$$scores_var.type", "quiz"] }
                        ]
                    }
                }
            }
        }
    },
    {
        $unset: ["scores", "student_id"] // 해당 필드를 제거한다.
    },
    {
        $unwind: "$tmp_scores"
    },
    {
        $group: {
            _id: "$class_id",
            exam_scores: {
               $push: {
                    $cond: {
                        if: {
                            $eq: ["$tmp_scores.type", "exam"]
                        },
                        then: "$tmp_scores.score", // 조건에 성립
                        else: "$$REMOVE"
                    }
               }
            },
            quiz_scores: {
               $push: {
                    $cond: {
                        if: {
                            $eq: ["$tmp_scores.type", "quiz"]
                        },
                        then: "$tmp_scores.score", // 조건에 성립
                        else: "$$REMOVE"
                    }
               }
            }
        }
    },
    {
        $project: {
            _id: 1,
            scores: {
                $objectToArray: { // 아래의 object를 array로 변환해준다.
                    exam: {
                        $avg: "$exam_scores"
                    },
                    quiz: {
                        $avg: "$quiz_scores"
                    }
                }
            }
        }
    },
    {
        $sort: {
            _id: -1
        }
    },
    {
        $limit: 5
    }
]);
```

## 참고

- [Aggregation Stages](https://www.mongodb.com/docs/manual/reference/operator/aggregation-pipeline/)
