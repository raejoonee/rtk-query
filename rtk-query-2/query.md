---
description: RTK Query에서 데이터 가져오기를 위해 사용합니다.
---

# Query

## 개요

Query\(이하 '쿼리'\)는 RTK Query를 사용할 때 가장 흔하게 사용되는 유즈 케이스입니다. 쿼리는 여러분이 원하는 대로 모든 데이터 가져오기 관련 라이브러리 작업을 수행할 수 있지만, 데이터를 읽을 때만 사용하는 것을 강력히 추천드립니다. _\(역주: GET 방식의 요청에만 사용하는 것을 추천한다는 뜻입니다.\)_ **서버에 있는 데이터를 변경하거나 데이터가 캐싱된 내용을 무효화할 가능성이 있는 작업의 경우 쿼리 대신 뮤테이션을 사용하세요.**

기본적으로 RTK Query의 쿼리는 `fetchBaseQuery` 와 함께 사용됩니다. `fetchBaseQuery` 는 axios처럼 흔하게 사용되는 다른 라이브러리들과 비슷한 방식으로 요청 헤더와 응답 파싱을 자동으로 처리하는 경량화된 [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) wrapper입니다. `fetchBaseQuery` 의 기본 옵션이 마음에 들지 않는다면 [쿼리를 커스터마이징하는 방법](https://redux-toolkit.js.org/rtk-query/usage/customizing-queries)에 대해 알아보세요.

여러분이 작업 중인 환경에 따라 `fetchBaseQuery` 또는 `fetch` 를 사용하기 위해서 `fetch` 를 `node-fetch` 또는 `cross-fetch` 등으로 폴리필해야 할 수도 있습니다. `useQuery` 의 사용법과 더 자세한 정보를 알아보려면 [이 곳](https://redux-toolkit.js.org/rtk-query/api/created-api/hooks#usequery)을 참조하세요.

## 쿼리 엔드포인트 정의하기

쿼리 엔드포인트는 `createApi` 의 `endpoints` 내부에서 객체를 반환하고 필드를 `builder.query()` 메소드를 사용하여 정의하는 방식으로 정의합니다. 또한 쿼리 파라미터를 포함하여 URL을 구성하는 `query` 콜백 함수 하나를 정의해야 합니다. 만약 임의의 비동기 로직을 처리하고 결과를 반환해야 한다면 [`queryFn` 콜백 함수](https://redux-toolkit.js.org/rtk-query/usage/customizing-queries#customizing-queries-with-queryfn)로 이를 대체할 수 있습니다.

만약 `query` 콜백 함수가 URL을 구성하기 위해 추가적인 데이터가 필요하다면, 하나의 인자를 받는 형식으로 작성해도 좋습니다. 단, 2개 이상의 매개변수를 전달해야 한다면 하나의 객체로 만들어 전달하세요.

쿼리 엔드포인트는 요청에 따른 결과가 캐시되기 전에 응답 내용을 수정할 수 있고, 캐시 무효화를 식별하기 위해 태그를 정의할 수 있고, 캐시 항목이 추가 및 제거될 때마다 추가 로직을 실행하기 위해서 캐시 라이프사이클 콜백 함수를 제공할 수 있습니다.

```typescript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query';
import { Post } from './types';

const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: '/',
  }),
  tagTypes: ['Post'],
  endpoints: (build) => ({
    getPost: build.query<Post, number>({
      // `query` 대신 `queryFn`을 사용할 수 있습니다.
      query: (id) => ({ url: `post/${id}` }),
      providesTags: (result, error, id) => [{ type: 'Post', id }],
      // 두번째 매개변수는 `QueryLifecycleApi`가 구조 분해 할당된 것입니다.
      async onQueryStarted(
        arg,
        {
          dispatch,
          getState,
          extra,
          requestId,
          queryFulfilled,
          getCacheEntry,
          updateCachedData,
        }
      ) {},
      // 두번째 매개변수는 `QueryCacheLifecycleApi`가 구조 분해 할당된 것입니다.
      async onCacheEntryAdded(
        arg,
        {
          dispatch,
          getState,
          extra,
          requestId,
          cacheEntryRemoved,
          cacheDataLoaded,
          getCacheEntry,
          updateCachedData,
        }
      ) {},
    }),
  }),
});
```

## React Hook을 써서 쿼리 실행하기

React Hook을 사용하면, RTK Query가 제공하는 유용한 추가 기능들을 사용할 수 있습니다. 이를 통해 얻을 수 있는 가장 큰 이득은 렌더링에 최적화된 hook을 사용할 수 있게 된다는 것입니다. 심지어 이 hook은 다양한 기능을 편하게 구현할 수 있도록 돕는 여러 불리언 값을 제공합니다.

모든 hook들은 여러분이 정의한 `endpoint` 의 이름에 기반하여 자동으로 생성됩니다. 예를 들면 엔드포인트를 `getPost: builder.query()` 와 같이 정의했다면 `useGetPostQuery` 라는 이름을 가진 hook이 자동으로 생성됩니다.

### Hook 타입

쿼리와 관련된 hook은 다음과 같이 총 5개가 존재합니다:

1. `useQuery` 
2. `useQuerySubscription`
3. `useQueryState`
4. `useLazyQuery`
5. `useLazyQuerySubscription`

실제로 거의 모든 경우 `useQuery` 를 기반으로 한 hook\(위에서 자동으로 생성된 `useGetPostQuery` 등을 말하는 겁니다.\)을 사용하여 구현하고자 하는 기능을 만들 수 있습니다. 다만, 아주 특별한 유즈 케이스를 위해 다른 훅을 사용할 수도 있습니다.

### 쿼리 Hook 옵션

쿼리 hook은 다음 두 가지 매개변수를 받습니다: `(queryArg?, queryOptions?)`

`queryArg` 는 URL을 생성하기 위해서 쿼리 콜백 함수를 지나가게 됩니다. 이 때 내부적으로 `useEffect` 의 종속성 배열로 전달됩니다. RTK Query는 값에 따른 차이를 비교하기 위해 `shallowEquals` 를 실행하는 방법을 최대한 유지하도록 하지만, 깊은 비교가 필요한 객체를 인자로 넘기는 경우 인자가 불변하지 않도록 `useMemo` 등을 써야 합니다.

`queryOptions` 객체는 여러분만의 데이터 가져오기 로직을 커스터마이징하기 위해서 사용할 수 있는 여러 추가 행동을 여러 개의 매개변수를 통해 받아옵니다.

* `skip`: 해당 렌더링 중 쿼리 동작을 '스킵'할 수 있습니다. 기본값은 `false`
* `pollingInterval`: 제공된 간격\(interval\)마다 자동으로 데이터를 다시 가져오도록 합니다. 단위는 밀리세컨드\(ms\)이며 기본값은 `0` 입니다.
* `selectFromResult`: hook을 통해 반환된 값을 결과의 하위 집합을 얻기 위해 변경할 수 있도록 합니다. 이 하위 집합은 렌더링에 최적화되어 있습니다.
* `refetchOnMountOrArgChange`: \(`true` 값이 제공될 경우\) 쿼리가 마운트될 때마다 항상 데이터를 다시 가져오도록 강제합니다. \(`number` 타입의 변수가 제공될 경우\) 같은 캐시를 사용하는 쿼리가 데이터를 다시 가져온 뒤 충분한 시간_\(역주: 제공된 숫자를 초\(second\)로 환산한 시간입니다.\)_이 흐른 뒤 데이터를 다시 가져오도록 강제합니다. 기본값은 `false` 입니다.
* `refetchOnFocus`: 브라우저 윈도우가 사용자에 의해 다시 포커스될 때마다 데이터를 다시 가져오도록 강제합니다. _\(역주: SWR에서 기본으로 제공하는 기능입니다.\)_ 기본값은 `false` 입니다.
* `refetchOnReconnect`: 네트워크 연결이 다시 성립됐을 때 데이터를 다시 가져오도록 강제합니다. 기본값은 `false` 입니다.

여기서 설정한 "데이터 다시 가져오기"와 관련된 모든 기능들은 `createApi` 에서 설정한 데이터를 덮어씁니다. 여기서 설정한 값이 우선순위가 높습니다.

### 자주 사용되는 쿼리 Hook 반환값

쿼리 hook은 요청의 현재 생명 주기 상태에 따라 상태 불리언 값을 반환하기도 하고, 쿼리 요청에 따른 가장 최근의 `data` 를 반환하기도 합니다. 이 모든 값이 프로퍼티의 형태로 들어있는 객체를 반환하는 형태로 말입니다. 다음은 여러분이 가장 자주 사용하게 될 프로퍼티들입니다. _\(역주: 대부분의 경우 객체 비구조화 할당의 방법으로 원하는 프로퍼티만 추출해서 사용하게 됩니다.\)_

* `data`: 쿼리가 반환한 값입니다.
* `error`: 쿼리를 실행하던 도중 에러가 발생했다면, 에러에 대한 정보입니다.
* `isUninitialized`: `true` 라면, 쿼리가 아직 시작하지 않았다는 뜻입니다.
* `isLoading`: `true` 라면, 쿼리가 아직 로딩 중이며 `data` 가 없다는 뜻입니다. 첫 요청이 시작될 때 `true` 값을 가지지만, 후속 요청이 시작될 땐 그렇지 않습니다.
* `isFetching`: `true` 라면, 쿼리가 현재 데이터를 가져오는 중이며, 요청에 대한 `data` 를 아직 가져오지 못했을 수도 있다는 뜻입니다. 첫 요청과 후속 요청 모두에 대해서 항상 `true` 값을 가집니다.
* `isSuccess`: `true` 라면, 쿼리가 요청을 성공적으로 마쳤고 `data` 를 가지고 있다는 뜻입니다.
* `isError`: `true` 라면, 쿼리가 `error` 상태에 있다는 뜻입니다.
* `refetch`: 쿼리를 강제로 다시 시작하도록 하는 함수입니다.

대부분의 경우 여러분들은 쿼리가 가져온 값을 읽기 위해 `data` 변수를 사용하고, 로딩 상태를 관리하기 위해 `isLoading` 과 `isFetching` 값을 사용하게 됩니다.

### 쿼리 Hook 사용 예제

```typescript
export const PostDetail = ({ id }: { id: string }) => {
  const { data: post, isFetching, isLoading } = useGetPostQuery(id, {
    pollingInterval: 3000,
    refetchOnMountOrArgChange: true,
    skip: false,
  });

  if (isLoading) return <div>로딩 중이에요...</div>;
  if (!post) return <div>아무 글도 없어요!</div>;

  return (
    <div>
      {post.name} {isFetching ? '데이터를 다시 가져오는 중...' : ''}
    </div>
  );
}
```

1. **첫 로딩 중**에는 화면에 '로딩 중이에요...'만 보입니다.
   * **첫 로딩**이란, 쿼리가 아직 pending 상태고, 캐싱된 데이터도 없는 상황입니다.
2. 요청이 `pollingInterval` 에 의해 다시 실행될 때마다, `post.name` 옆에 '데이터를 다시 가져오는 중...'이 보입니다.
3. 만약 사용자가 이 `PostDetail` 컴포넌트를 닫았다가 허용된 시간 내에 다시 열었을 경우, 캐싱된 데이터가 즉각 보여지게 되고 기존에 명시한 행동을 스스로 알아서 다시 이어서 시작합니다.

### 쿼리 로딩 상태

React에 최적화된 버전_\(역주: '@reduxjs/toolkit/query/react'의 경로를 통해 `import` 되었다는 뜻입니다.\)_의 `createApi` 메소드를 통해서 자동으로 생성된 `useQuery` hook은 주어진 쿼리의 현재 상태를 반영하는 불리언 값을 제공합니다.

RTK Query는 유도된 정보를 제공하는 데 더 큰 유연성을 제공하기 위해서 쿼리 엔드포인트에 대해  `isLoading` 과 `isFetching` 을 의미론적으로 구별합니다. 

* `isLoading` 은 주어진 hook의 쿼리가 _처음으로_ 전송되는 중인지 알려줍니다. 이 동안은 사용 가능한 데이터가 없습니다.
* `isFetching` 은 주어진 엔드포인트로 쿼리와 파라미터가 전송되는 중인지 알려줍니다. 단, _처음으로_ 전송하는 경우는 아닐 수 있습니다. 이 hook에 의하여 전송된 요청이 이미 존재하는 경우가 있을 수 있고, 이 경우 사용 가능한 데이터가 있습니다.

이 의미 구별은 개발자가 UI 행동을 처리하는 데 더 적절한 방법을 사용할 수 있도록 선택지를 제공합니다. 예를 들면, 처음 로딩이 일어나는 동안 스켈레톤을 보여주기 위해 `isLoading` 을 사용할 수 있고, 사용자가 첫 번째 페이지에서 두 번째 페이지로 넘어갈 때 기존에 캐싱된 데이터를 지우고 새로운 데이터를 가져오기 위해 `isFetching` 을 사용할 수 있게 됩니다.

```typescript
import { Skeleton } from './Skeleton';
import { useGetPostsQuery } from './api';

function App() {
  const { data = [], isLoading, isFetching, isError } = useGetPostsQuery();

  if (isError) return <div>에러가 발생했습니다!</div>;

  if (isLoading) return <Skeleton />;

  return (
    <div className={isFetching ? 'posts--disabled' : ''}>
      {data.map((post) => (
        <Post
          key={post.id}
          id={post.id}
          name={post.name}
          disabled={isFetching}
        />
      ))}
    </div>
  );
}
```

### 쿼리 캐시 키

여러분이 쿼리를 실행할 때 RTK Query는 자동으로 요청에 대한 내부 `queryCacheKey` 를 생성하고 요청 파라미터를 직렬화합니다. 이 이후로 똑같은 `queryCacheKey` 를 생성하는 모든 요청들은 결과 데이터를 기존 데이터와 비교한 뒤 불필요한 중복 데이터를 생성하지 않고 기존의 데이터를 제거합니다. 그리고 만약 해당 쿼리를 구독하고 있는 컴포넌트에 의해 `refetch` 가 발생할 경우 업데이트된 내용을 모두 공유하게 합니다.

### 쿼리 결과로부터 데이터 얻기

쿼리를 구독하고 있는 부모 컴포넌트가 있는 상황에서 해당 쿼리에 있는 아이템 하나를 자식 컴포넌트에서 사용해야 할 때가 있습니다. 대부분의 경우 이미 요청에 대한 응답을 보유한 상황에서 추가 요청을 만들고 싶지는 않을 겁니다.

`selectFromResult` 를 사용하면 애플리케이션의 퍼포먼스를 저해하지 않으면서 쿼리 결과의 특정 세그먼트를 가져올 수 있습니다. 이 기능을 사용할 때 선택한 항목의 기본 데이터가 변경되지 않으면 구성 요소가 다시 렌더링되지 않습니다. 만약 선택한 항목이 더 큰 Collection\(Array, Map, Set 등\)의 한 요소인 경우, 동일한 Collection의 요소 변경 사항을 무시합니다.

```typescript
function PostsList() {
  const { data: posts } = api.useGetPostsQuery();

  return (
    <ul>
      {posts?.data?.map((post) => (
        <PostById key={post.id} id={post.id} />
      ))}
    </ul>
  );
}

function PostById({ id }: { id: number }) {
  // 주어진 id를 통해 post를 선택하고,
  // 주어진 post의 data가 바뀔 때만 렌더링이 다시 일어납니다.
  const { post } = api.useGetPostsQuery(undefined, {
    selectFromResult: ({ data }) => ({
      post: data?.find((post) => post.id === id),
    }),
  });

  return <li>{post?.name}</li>;
}
```

### 데이터를 다시 가져오도록 강제하기

기본적으로 이미 존재하는 쿼리와 똑같은 쿼리를 만드는 컴포넌트를 추가하는 것은 새 요청을 만들지 않습니다. 그 요청에 대한 데이터는 이미 캐싱되어 있을 것이고, 캐싱된 데이터를 사용하면 되기 때문입니다. 특정 상황에서 이런 특성을 무시하고 데이터를 다시 가져오도록 강제해야 한다면 hook이 반환하는 `refetch` 함수를 호출하면 됩니다.

만약 여러분이 React를 사용하고 있지 않거나 hook을 사용하기 싫다면, 다음과 같은 방법으로 `refetch` 에 접근할 수 있습니다.

```typescript
const { status, data, error, refetch } = dispatch(
  pokemonApi.endpoints.getPokemon.initiate('bulbasaur')
);
```

## 예제: 중복 요청 제거, 캐싱

이 예제는 같은 엔드포인트를 향한 중복된 요청을 어떻게 처리하는지, 어떤 방식으로 캐싱을 진행하는지 확인할 수 있는 예제입니다.

1. 첫 번째 `Pokemon` 컴포넌트가 마운트되며 그 순간 즉시 'bulbasaur'_\(이상해씨\)_의 데이터를 가져옵니다.
2. 1초 가량이 지난 뒤, 다른 `Pokemon` 컴포넌트가 'bulbasaur'_\(이상해씨\)_를 렌더링합니다.
   * 'Loading...' 메시지를 보이지도 않고, 개발자 도구를 통해 확인해 보면 새 네트워크 요청을 생성하지도 않는다는 것을 확인할 수 있습니다. 즉, 2번 과정에서 캐싱된 데이터를 사용한다는 뜻입니다!
3. 다음으로 'pikachu'_\(피카츄\)_를 렌더링하는 `Pokemon` 컴포넌트가 생기고, 새 네트워크 요청이 발생합니다.
4. 여러분이 `Refetch` 버튼을 클릭하면, 해당 포켓몬의 데이터를 렌더링하고 있는 모든 컴포넌트가 단 한 번의 요청으로 모두 업데이트됩니다.

{% embed url="https://codesandbox.io/embed/github/reduxjs/redux-toolkit/tree/master/examples/query/react/deduping-queries?autoresize=1&fontsize=12&hidenavigation=1&theme=dark" %}

`Add bulbasaur` 버튼을 클릭해서 이상해씨를 더 만들어보세요. 새로 생성된 이상해씨를 렌더링하고 있는 컴포넌트도 기존의 이상해씨를 렌더링하고 있는 컴포넌트와 함께 동시에 데이터를 갱신하는 것을 확인할 수 있을 것입니다.

