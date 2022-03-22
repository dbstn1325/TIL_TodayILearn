# error 접근

- .catch() 부문에서 err에 대한 데이터에 접근하기 위한 코드
    - err.data: 에러의 데이터 확인
    - err.response : 요청이 성공적이며, 서버가 2xx의 범위를 벗어나는 상태 코드로 응답
    - err.request: 요청이 이루어 졌으나 응답을 받지 못함, 브라우저의 XMLHttpRequest 인스턴스 또는 Node.js의 http.ClientRequest 인스턴스
    - err.message: 오류를 발생시킨 요청을 설정하는 중에 문제가 발생
    - err.config.validateStatus: 사용자 정의 HTTP상태 코드 오류 범위 정의 가능


**실제 썼던 예시**
```jsx
const handleSearch = async () => {
    console.log(searchName);
    console.log("search : " + searchName);

    await axios.get(`http://localhost:3001/mypage/friendList/edit/${searchName}`)
    .then((e) => {
      setFriend(e.data)
    })
    .catch((err) => {
      if(err.response.status === 404)
        setFriend("존재하지 않는 유저"); 
      });
  };
```

1. **err.response.data**


![](https://images.velog.io/images/dbstn1325/post/2524e689-134e-431d-a6b1-6dce37ea6786/image.png)

2. **err.response.status**


![](https://images.velog.io/images/dbstn1325/post/a701df06-db9f-4ca6-9a47-4db5a7b36f7d/image.png)


---

# 문제 상황

### 클라이언트 쪽

- 클라이언트에서 @Param()을 통해 “정윤수”라는 데이터(한글)을 담아, 서버쪽으로 보낼려고 한다. 하지만, 이 과정 중 인코딩 상황이 일어났다. 이는 대부분 Content-Type을 charset=utf-8로 수정하여 보내면 해결된다. 하지만, 클라이언트쪽의 Params 데이터를 확인해보니, 정상적으로 utf-8형태로 보내고 있었고, 이는 또 다른 문제라고 생각해 이에 대한 기록을 남긴다.

![](2022-03-22-17-05-23.png)

![](2022-03-22-17-05-39.png)

### 서버 쪽

- 클라이언트 쪽에서 인코딩현상이 발생한 데이터가 넘어와, 정상적으로 서버쪽에서 데이터를 확인할 수 없었다.

![](https://images.velog.io/images/dbstn1325/post/525f7b86-50e2-4dc2-aca0-54fa076892d7/image.png)

![](https://images.velog.io/images/dbstn1325/post/237a36c8-059e-4070-9868-41aaea382887/image.png)

## 서버 코드 확인 (users.service.ts)

- 서버쪽 service부분에서 클라이언트로부터 넘어온 데이터를 decode하려 했다. 하지만 에러는 고쳐지지 않았고, 이로써 서버쪽 에서 문제가 아니라, 클라이언트 쪽에서 넘어오는 데이터의 문제라고 판단할 수 있었다.

![](https://images.velog.io/images/dbstn1325/post/81cf5c92-d17a-4375-a6db-fb5c63cba6c3/image.png)


![](https://images.velog.io/images/dbstn1325/post/994fb1ee-8367-46d3-b208-7498cbfaaf8e/image.png)
## 원인 분석: 클라이언트 문제

- 클라이언트 쪽의 코드를 살펴보니, friendName을 내가 문자열로 한 번 더 감싸주고 있었다. 문자열 자체를 넘겨주니, axios get 요청을 보낼 때, 인코딩이 발생했던 것이다.

```jsx
export function OptionBox() {

const handleSearch = async() => {
      const searchName = JSON.stringify(friendName);
			const searchData = await axios.get(`http://localhost:3001/mypage/friendList/edit/${searchName}`);
      
      //서버쪽에서 디코딩 처리 후 result 출력
      await axios.get(`http://localhost:3001/mypage/friendList/edit/${searchName}`)
        .then((result) => { return console.log(result); });
			
    }

return ( 
			<div>
					<form>
            <div className="optionBox_inner">
              <input type="text" value={friendName} onChange={onChange} name="name"></input>
              <FontAwesomeIcon onClick={handleSearch} type="submit" icon={faPropIcon} className="search" />
            </div>
          </form>
			</div>
)

}
```

## 해결 코드: 클라이언트 수정

- 해당 코드를 지우고, 서버 쪽에서 해당 데이터를 디코드 해주면서 해결되었나 했다.

```jsx
const handleSearch = async() => {
      console.log(friendName);
      console.log("search : " + friendName);
      
      const searchData = await axios.get(`http://localhost:3001/mypage/friendList/edit/${friendName}`);
      console.log(searchData);
    }
```

### 놀라운 점 (users.service.ts)

- 어찌보면 당연한 일이었다. 클라이언트쪽에서 이미 utf-8로 데이터를 전달해주었기 때문에, 해당 한글로 된 이름은 인코딩될 일이 없었고, 서버 쪽에서는 decode코드를 굳이 쓰지 않아도 해당 데이터가 잘 받아올 수 있었다. 넘어오는 데이터를 하나씩 확인해가며 코드는 더욱 이전보다 더 깔끔해졌다.

```jsx
getUserById(name: string): Promise <String> {
        return this.userRepository.getUserById(name);
    }
```

