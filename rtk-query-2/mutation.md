---
description: >-
  RTK Query에서 데이터 갱신 요청을 서버에 보내고 데이터 변경점을 로컬 캐시에 반영하기 위해 사용합니다. 데이터를 다시 가져오도록
  강제하며 캐싱된 데이터를 무효화합니다.
---

# Mutation

## Mutation 엔드포인트 정의하기

Mutation\(이하 '뮤테이션'\) 엔드포인트는 `createApi` 의 `endpoints` 내부에서 객체를 반환하고 필드를 `builder.mutation()` 메소드를 사용하여 정의하는 방식으로 정의합니다. 또한 쿼리 파라미터를 포함하여 URL을 구성하는 `query` 콜백 함수 하나를 정의해야 합니다. 만약 임의의 비동기 로직을 처리하고 결과를 반환해야 한다면 [`queryFn` 콜백 함수](https://redux-toolkit.js.org/rtk-query/usage/customizing-queries#customizing-queries-with-queryfn)로 이를 대체할 수 있습니다. `query` 콜백 함수는 URL, 사용할 HTTP 메소드, request body를 포함하고 있는 객체를 반환해야 합니다.

만약 `query` 콜백 함수가 URL을 구성하기 위해 추가적인 데이터가 필요하다면, 하나의 인자를 받는 형식으로 작성해도 좋습니다. 단, 2개 이상의 매개변수를 전달해야 한다면 하나의 객체로 만들어 전달하세요.

뮤테이션 엔드포인트는 요청에 따른 결과가 캐시되기 전에 응답 내용을 수정할 수 있고, 캐시 무효화를 식별하기 위해 태그를 정의할 수 있고, 캐시 항목이 추가 및 제거될 때마다 추가 로직을 실행하기 위해서 캐시 라이프사이클 콜백 함수를 제공할 수 있습니다.

```typescript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query';
import { Post } from './types';

const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: '/',
  }),
  tagTypes: ['Post'],
  endpoints: (build) => ({
    updatePost: build.mutation<Post, Partial<Post> & Pick<Post, 'id'>>({
      // `query` 대신 `queryFn`을 사용할 수 있습니다.
      query: ({ id, ...patch }) => ({
        url: `post/${id}`,
        method: 'PATCH',
        body: patch,
      }),
      invalidatesTags: ['Post'],
      // onQueryStarted는 업데이트 최적화에 유용하게 사용할 수 있습니다.
      // 두번째 매개변수는 `MutationLifecycleApi`가 구조 분해 할당된 것입니다.
      async onQueryStarted(
        arg,
        { dispatch, getState, queryFulfilled, requestId, extra, getCacheEntry }
      ) {},
      // 두번째 매개변수는 `MutationCacheLifecycleApi`가 구조 분해 할당된 것입니다.
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
        }
      ) {},
    }),
  }),
});
```

## React Hook을 써서 뮤테이션 실행하기

### 뮤테이션 Hook의 특성

`useQuery` 와 다르게, `useMutation` 은 튜플을 반환합니다. 튜플의 첫 번째 항목은 "트리거" 함수이며 두 번째 요소는 `status`, `error`, `data` 를 가지고 있는 객체를 포함합니다.

`useQuery` 와 다르게, `useMutation` 은 자동으로 실행되지 않습니다. 뮤테이션을 실행하기 위해선 앞서 말한 트리거 함수를 호출해야 합니다.

### 자주 사용되는 뮤테이션 Hook 반환값

앞서 말한 대로 `useMutation` Hook은 **트리거 함수**와 **뮤테이션 결과를 담고 있는 객체**를 튜플의 형태로 반환합니다.

**트리거 함수**는 호출됐을 때 해당 엔드포인트로 뮤테이션 요청을 보내는 역할을 합니다. 트리거 함수를 호출하면 `unwrap` 프로퍼티를 가지고 있는 프로미스가 반환됩니다. `unwrap` 은 뮤테이션 호출 내부 구조를 보여주고 RTK Query에 의해 가공되지 않은 원래 응답과 오류를 제공하기 위해 호출될 수 있습니다. 뮤테이션이 실행된 상황에서 오류가 발생했는지, 성공적으로 실행됐는지 여부를 판단할 때 유용하게 사용할 수 있습니다.

**뮤테이션 결과**는 요청의 현재 생명 주기 상태에 따라 상태 불리언 값을 반환하기도 하고, 뮤테이션 요청에 따른 가장 최근의 `data` 를 반환하기도 합니다. 이 모든 값이 프로퍼티의 형태로 들어있는 객체를 반환하는 형태로 말입니다. 다음은 여러분이 가장 자주 사용하게 될 프로퍼티들입니다.

* `data`: 트리거 함수의 응답이 가장 최근 반환한 값입니다. 같은 hook 인스턴스로부터 후속 트리거 함수를 호출하면, 이 값은 새 데이터를 받을 때까지 `undefined` 를 반환합니다. 이전 응답에 대한 데이터에서 새로운 데이터로 부드러운 전환이 발생하게 하려면 컴포넌트 수준에서 캐싱하는 것을 고려해보세요.
* `error`: 뮤테이션을 실행하던 도중 에러가 발생했다면, 에러에 대한 정보입니다.
* `isUninitialized`: `true` 라면, 뮤테이션이 아직 시작하지 않았다는 뜻입니다.
* `isLoading`: `true` 라면, 뮤테이션이 시작되었고 응답을 기다리는 중이란 뜻입니다.
* `isSuccess`: `true` 라면, 가장 최근에 시작된 뮤테이션이 요청을 성공적으로 마쳤고 `data` 를 가지고 있다는 뜻입니다.
* `isError`: `true` 라면, 가장 최근에 시작된 뮤테이션이 `error` 상태에 있다는 뜻입니다.

RTK Query에서 뮤테이션은 [쿼리와 다르게](https://raejoonee.gitbook.io/rtk-query/rtk-query-2/query#undefined-3) **'loading'**과 **'fetching'**의 의미를 구별하지 않습니다. 뮤테이션의 경우 후속 트리거 함수 호출이 반드시 앞선 트리거 함수 호출과 관련 있다고 여겨지지 않기 때문입니다. 따라서 뮤테이션은 **'데이터를 다시 가져온다'**는 개념 없이, **로딩 중** 또는 **로딩 중이 아님** 이 두 가지 상태만 존재합니다.

### 뮤테이션을 사용하는 예제

이 예제에선 게시글\(Post\) 데이터를 `useQuery` 를 사용해서 가져오고, `EditablePostName` 컴포넌트가 렌더링됩니다. 이 컴포넌트는 게시글의 이름을 바꾸는 기능을 제공합니다.

```typescript
export const PostDetail = () => {
  // 역주: React Router에서 전달받은 URL Parameter를 가져오기 위해 사용하는 Hook입니다.
  const { id } = useParams<{ id: any }>();
  
  const { data: post } = useGetPostQuery(id);

  const [
    updatePost, // 트리거 함수
    { isLoading: isUpdating }, // 뮤테이션 결과가 구조 분해 할당된 것입니다.
  ] = useUpdatePostMutation()

  return (
    <Box p={4}>
      <EditablePostName
        name={post.name}
        onUpdate={(name) => {
          // 만약 뮤테이션 결과에 즉시 접근하고 싶다면, unwrap() 체인을 사용해야 합니다.
          // 결과의 payload를 얻거나 에러를 catch하고 싶다면 다음 코드를 추가하세요.
          /*
            updatePost().unwrap()
              .then(fulfilled => console.log(fulfilled))
              .catch(rejected => console.error(rejected))
          */
          
          /*
            역주:
            에러 핸들링이 주 목적이라면,
            오류가 발생할 때 unwrap()이 에러를 throw하는 특성을 이용하여
            unwrap() 체인 대신 try-catch문을 사용할 수도 있습니다.
          */
          
          return (
            // id와 갱신된 name을 사용해서 트리거 함수를 실행합니다.
            updatePost({ id, name })
          )
        }}
        isLoading={isUpdating}
      />
    </Box>
  );
}
```

