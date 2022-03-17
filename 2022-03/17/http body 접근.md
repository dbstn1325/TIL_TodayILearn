# http body 데이터 가져오기

## 문제 상황
![](https://images.velog.io/images/dbstn1325/post/3bcb0382-ea7c-4666-86c5-9725b25b4e42/image.png)

![](https://images.velog.io/images/dbstn1325/post/662c0093-f3a2-4580-adac-bc4947c8dfa7/image.png)


클라이언트쪽에서 body에 email을 담아서 보낸다. 이 때, ‘꼭 Body('email')방법으로만 email을 가져올 수 있을까??’  라는 의문이 생겼다. 아래는 이에 대한 3가지 구현 방법을 소개한다.



### **클라이언트 (고정)**

```jsx
const handleFriendRequest = async () => {
    // setReqStatus("send");
    console.log("send");

    //requsetTo: "dbstn6477@gmail.com"이 들어있음
    await axios
      .post("http://localhost:3001/mypage/check/req", {email: requestTo}, {})
      .then((e) => {
        console.log(e);
      });
```

### **서버**

**방법 1. 아래와 같은 Object.entiries 메서드를 통해 body에서 이메일을 가져올 수도 있다!**

```jsx
@Post('/check/req')

checkUserFriendReq(@Body() email: string): any{

   for (const [key, value] of Object.entries(email)){
	console.log(`${key}: ${value}`);
   }

}
```

<방법 1의 실제 구현>

```jsx
@Post('/check/req')

checkUserFriendReq(@Body() email: string): any{
   for (const value of Object.entries(email)){
       email = value[1];
   }
}

```

**방법 2. email객체에서 email을 한번 더 불러옴으로써 Body()의 email에 접근 가능하다.** 

- 하지만 이 방법은 ts에 맞지 않는 방법이므로 추천하지 않는 방법이다.

```jsx
@Post('/check/req')
    checkUserFriendReq(@Body() email): any{
        console.log(email.email);
    }
```

**방법 3. Body(’emai’)로 접근**

- Body의 email이라는 key에 접근하는 방법이다. 가장 간단하고, 쉬운 방법이긴 하지만, 항상 익숙하고 쉬운 방법만을 고집하는 건 올바르지 않은 점을 기억하자.

```jsx
@Post('/check/req')
    checkUserFriendReq(@Body('email') email:string): any{
        console.log(email);
    }
```
