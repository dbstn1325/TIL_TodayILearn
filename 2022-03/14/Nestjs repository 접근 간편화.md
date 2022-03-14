# 03/14

기존에는 아래와 같은 2단계에 걸쳐, 특정 유저의 친구 정보에 접근했었다.

1. 특정 유저의 정보에 접근
2. 해당 유저의 친구 정보에 접근

해당 코드는 매우 비효율적으로 보였다. 이 코드는 실무에서 쓰일리가 없었고, 여러 자료들을 찾아보았다. 기존의 user Repository에서가 아닌, friend Repsitory에서의 단 한번의 접근으로 기존의 작업하던 2번의 과정을 1번으로 줄일 수 있었다.



# friend, friend_req Repository & Entity 수정

## 기존 코드
* User Repository에서 처리
* 2번의 DB 접근 발생

```jsx
//user.repository.ts
async getUserAllFriends(userId: number): Promise<any> {
    //유저 정보 검수과정 더 필요
    
    //유저의 정보 가져오기
    const user = await this.findOne({userId,});
    if (!user) {
       throw new NotFoundException('존재하지 않는 유저입니다.');
    }
		
    //유저의 친구목록 접근
    let friends = [];
    friend_list.forEach((user) => {
      //친구 한명당
      friends.push(user.friends);
    });
    return friends;
  }
```


## 변경 코드
* Friend Repository에서 처리
* 1번의 DB 접근으로 코드 간편화
```jsx
//friend.repository.ts
async getFriendByUser(userId: number) {
				//유저의 친구목록 접근
        const result = await this
            .createQueryBuilder('friend')
            .innerJoin('friend.user', 'user')
            .where('user.user_id = :userId', { userId })
            .getMany();
        //console.log(result);
        return result;
}
```

```jsx
//friend.repository.ts
async checkUserFriendReq(email: string) {
        const gmail = email + "@gmail.com";
        const result = await this
            .createQueryBuilder('received_req')
            .innerJoin('received_req.user', 'user')
            .where('user.email = :email', { email: gmail })
            .getMany();
        return result;
}
```


## Entity 수정 코드

```jsx
//user.entity.ts
@OneToMany(() => ReceivedReq, (received_req) => received_req.user, { eager: true })
    received_reqs?: ReceivedReq[];

  @OneToMany(() => Friend, (friend) => friend.user, { eager: true })
    friends?: Friend[];
```


```jsx
//friend.entity.ts
...
@ManyToOne(()=> User, (user) => user.friends, {
        eager: false 
    })
    user: User;
...
```


```jsx
//friendReq.entity.ts
...
@ManyToOne(() => User, (user) => user.received_reqs, {
        eager: false 
    })
    user: User;
...
```



