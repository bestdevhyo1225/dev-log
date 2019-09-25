## Mongoose Schema 사용자 정의 메소드 (statics vs methods)

<br>

Mongoose는 ODM(Object Document Mapper) 프레임워크이다. Mongoose는 스키마 클래스를 만들어서 사용자 정의 메소드를 만들 수 있다. 개인 프로젝트에서 사용자 정의 메소드를 사용하다가 궁금한 점이 생겼다. `statics`와 `methods`라는 두 가지 키워드로 정의할 수 있는데 두 개의 차이점이 무엇인지 궁금했고, 나중에 개발을 할 때 차이점을 기억하면서 적절하게 사용하고 싶었기 때문에 글을 정리하게 되었다.

<br>

### :book: statics & methods 키워드

나는 보통 Mongoose를 사용할 때 Schema 클래스를 만들고, 사용자 정의 함수를 만들어서 사용하곤 한다. 

```typescript
import mongoose from 'mongoose';

const userSchema = new Schema({
    id : String,
    pw : String,
    name : String,
});

userSchema.statics.test1 = async function(user: any) {

}

userSchema.methods.test2 = async function(user: any) {

}

export const userSchema = mongoose.models.userSchema || mongoose.model('User', userSchema);
```

`statics`

* Java의 static 메소드와 동일하다고 보면 된다. 즉, 객체를 생성하지 않고 바로 컬렉션에 있는 데이터에 접근할 수 있다.
    
* 데이터를 조회하는 기능을 구현할 때 사용하면 된다.

```typescript
// ID로 유저를 조회한다.
userSchema.statics.findUser = async function(id: string) {
    return this.findOne({ id : id });
}
```

<br>

`methods`

* Java에서 일반 메소드와 동일하다고 생각한다. 

* **모델의 객체 인스턴스가 살아있을 때,** methods 키워드로 되어있는 함수를 호출할 수 있다.

```typescript
// models/user.ts
userSchema.methods.verify = async function(pw: string) {
    return this.pw === pw;
}

userSchema.statics.findOneByUserID = async function(id: string) {
    return this.findOne({ id });
}
```

```typescript
// controllers/user.ts
import User from './model/user.ts'

export const login = async (req: Request, res: Response) => {
    try {
        const { id, pw } = req.body;
        const user: any = await User.findOneByUserID(id);
        if (!user || !user.verify(pw)) {    // user 인스턴스의 verify 함수를 호출
            // 로그인 실패
        }
    } catch (error) {
        console.log(error);
        // 생략..
    }
}
```

<br>

### :book: 정리

* `methods`는 하나의 `Document(Record)` 단위로 사용하는 기능이다.

* `statics`는 하나의 `Collection(Table)` 단위로 사용하는 기능이다.

<br>

### :book: 참고

* [mongoose 간단 예제와 methods, statics 활용 차이](http://kese111.blogspot.com/2015/01/mongoose-methods-statics.html)