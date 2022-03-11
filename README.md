# -thinking-about-useNavigate

### useNavigate의 비효율적 랜더링
react 강의를 제작하다가 알게 된 사실\
useNavigate는 기본적으로 effect hook을 사용한다.\
따라서 useNavigate를 사용하게 되면 React.memo로 memorizing을 하더라도 

```
const navigate = useNavigate();
```

가 컴포넌트 안에 있으면 무조건 다시 랜더하게 된다. 이에 대한 증거 자료는 아래와 같다.



```
const ShowPostList = ({  }) => {
  const [loading, setLoading] = useState(true);
  const [isPost, setIsPost] = useState(false);
  const [postList, setPostList] = useState([]);

  // const navigate = useCallback(useNavigate, []);
  const navigate = useNavigate();

  const goWrite = useCallback(() => {
    navigate(-2);
  }, []);
  
  ...
  export default React.memo(ShowPostList);
```

위와 같은 형식으로 컴포넌트를 구성할 경우, 

---
![image](https://user-images.githubusercontent.com/59525356/157872277-75bc9dbd-ae3e-4305-a745-caa0e819788b.png)
---

memo 처리된 ShowPostList가 다시 랜더된다. 이유는? Hook 5이다. 

![image](https://user-images.githubusercontent.com/59525356/157872512-1b17b76e-2f26-4b1e-8e24-181709e94236.png)
---

ShowPostList의 Hook 5는 Location의 Effect이고 이는 navigation의 Effect로 추정된다.

### 해결

이에 대한 고민은 나만 한 것은 역시 아니었다.\
관련 토의 내용은 아래에 정리 되어 있다.
https://github.com/remix-run/react-router/issues/7634

---
위 토의에서 찾을 수 있던 해법은 부모 컴포넌트에서 useCallback까지 써서 주는 것이다.

```// child
export const DayChart = memo(function DayChart ({
  dayData,
  navigateToDay,
}: {
  dayData: DayData;
  navigateToDay: (dayId: string) => void;
}) {
  return (
    <div onClick={() => navigateToDay(dayData.dayId)}>
      <Chart data={dayData} />
    </div>
  );
});

// parent
function DaysList() {
  const { days } = useAPI(...)
  const navigate = useNavigate();

  const navigateToDay = useCallback(
    (dayId: string): void => {
      navigate(`/my-route/${dayId}`);
    },
    [navigate]
  );

  return <div>{days.map((day) => <DayChart dayData={day} navigateToDay={navigateToDay} />)}</div>;
}
```

확실히 쓸만한 방법이지만, 모든 navigate관련 함수를 APP에서 관리해야 한다는 문제점이 있다.\
이에 대해 더 좋은 해결책이 없을까 고민이다.
