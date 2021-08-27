---
description: 'RTK Query의 기본 개념을 소개하고, RTK Query를 어떻게 사용할 수 있는지 간략하게 안내합니다.'
---

# RTK Query 소개

**RTK Query**는 데이터 가져오기\(fetching\)와 캐싱하기\(caching\)를 위한 강력한 도구입니다. **데이터를 가져오고 캐싱하는 로직을 직접 일일히 번거롭게 작성하는 과정을 없애고,** 웹 애플리케이션이 데이터를 불러오는 보편적인 과정을 단순화하기 위해 탄생했습니다.

RTK Query는 **Redux Toolkit 패키지에 포함된 선택적 애드온**이며, Redux Toolkit에 있는 다른 API들의 위에 기능이 구현되어 있으므로 함께 사용할 수 있습니다.

## 동기

웹 애플리케이션은 보통 데이터를 화면에 보여주기 위해 서버로부터 그 데이터를 가져올 필요가 있습니다. 또한, 그 데이터를 업데이트하고, 업데이트된 데이터를 서버로 보내고, 서버에 있는 데이터와 동기화한 클라이언트 측 데이터를 캐싱하여 보관할 필요도 있습니다. 요즘의 웹 애플리케이션들은 다음과 같은 많은 추가 기능들을 구현할 필요가 있기 때문에 이 과정들은 너무 복잡해지기 마련입니다:

* 스피너를 보여주기 위해 로딩 상태를 추적해야 합니다.
* 같은 데이터를 가져오기 위해 중복된 요청을 보내지 말아야 합니다.
* 사용자가 UI와 상호작용하고 있을 때 캐시 수명을 관리해야 합니다.
* UI가 빠르다고 느껴지도록 하기 위해 업데이트 과정을 최적화해야 합니다.

  Redux에는 위와 같은 유즈 케이스를 해결하기 위해 도와주는 추가 기능이 내장되어 있지 않습니다. Redux는 항상 최소한의 도움만 제공하고 실제 로직을 작성하는 것은 모두 개발자가 직접 해결해야 합니다. 이 과정을 보다 더 간편하게 진행할 수 있도록 Redux 공식 문서에서 [로딩 상태와 요청 결과를 추적하기 위한 전체 요청 과정에서 액션을 디스패치하는 과정](https://redux.js.org/tutorials/fundamentals/part-7-standard-patterns#async-request-status)을 상세히 가르쳐주고 있고, 이 전형적인 패턴을 추상화하고자 [Redux Toolkit의 `createAsyncThunk` API](https://redux-toolkit.js.org/api/createAsyncThunk)가 생겨났습니다. 그러나 많은 개발자들이 여전히 로딩 상태와 캐싱된 데이터를 관리하기 위해 수많은 리듀서 로직을 직접 작성하고 있습니다.

지난 2년 간 React 커뮤니티는 **"데이터 가져오기와 캐싱하기"는 "상태 관리"와 완전히 다른 관심사**라는 사실을 깨닫게 되었습니다. 여러분은 지금까지 그래왔듯이 앞으로도 데이터를 캐싱하기 위해 Redux와 같은 상태 관리 라이브러리를 사용할 수도 있지만, 데이터를 가져오는 상황을 해결하기 위한 목적으로 만들어진 도구를 사용하는 것이 더 나을 것입니다.

RTK Query는 이런 상황에서 해결법을 제시한 다른 선구자들\(Apollo Client, React Query, Urql, SWR\)로부터 영감을 받아 탄생했습니다. 그러나 그들과는 다르게 API 디자인 방식에 다음과 같은 독특한 접근법들을 추가했습니다:

* Redux Toolkit의 `createSlice`와 `createAsyncThunk` API 위에 기능이 구현되어 있습니다.
* Redux Toolkit이 모든 UI 레이어에서 사용될 수 있기 때문에, RTK Query의 기능 또한 모든 UI 레이어에서 사용될 수 있습니다.
* API 엔드포인트는 인자로부터 쿼리 파라미터를 생성하는 방식과 캐싱을 위해 응답을 변환하는 방식을 포함하여 미리 정의됩니다.
* RTK Query는 데이터를 가져오는 모든 과정을 캡슐화한 React Hooks를 생성할 수 있습니다. 이 Hooks는 컴포넌트에게 `data` 필드와 `isLoading` 필드를 제공하며 컴포넌트가 마운트되고 언마운트되는 동안 캐싱된 데이터의 수명을 관리합니다.
* RTK Query는 첫 데이터를 가져온 뒤 웹소켓 메시지를 통해 스트리밍되는 캐시 업데이트와 같은 유즈 케이스를 해결할 수 있는 "cache entry lifecycle" 옵션을 제공합니다.
* OpenAPI와 GraphQL 스키마에서 API slice를 만드는 예제 코드를 제공합니다.
* 마지막으로, RTK Query는 완전히 TypeScript를 기반으로 작성되어 완벽한 TypeScript 사용 경험을 제공합니다.

## 포함 기능

### API

RTK Query는 다음과 같은 API를 포함하고 있습니다:

* `createApi()`: RTK Query 기능의 핵심입니다. 일련의 엔드포인트들로부터 데이터를 검색할 방법을 표현하는 엔드포인트 집합을 정의합니다. 해당 엔드포인트들로부터 데이터를 검색할 방법을 설정할 수 있고 어떻게 데이터를 가져오고 변형할 지에 대해 설정할 수 있습니다. 대부분의 경우, "base URL 하나에는 하나의 API slice"라는 관습적 규칙에 따라 애플리케이션 당 한 번 사용하게 될 것입니다.
* `fetchBaseQuery()`: 요청을 단순화하기 위한 목적으로 사용하는 wrapper입니다. 대부분의 경우, `createApi` 내부에서 사용하도록 권장됩니다.
* `ApiProvider`: **Redux 저장소가 없는 경우**, `Provider`처럼 사용할 수 있습니다. **이미 사용 중인 Redux 저장소가 있는 경우 서로 충돌이 일어날 수 있습니다.**
* `setupListeners()`: `refetchOnMount`와 `refetchOnReconnect`를 사용할 수 있도록 합니다.

## 기본 사용법

### API slice 만들기

RTK Query는 Redux Toolkit 패키지 설치 과정에서 함께 설치됩니다. 다음과 같은 두 엔트리 포인트를 통해 사용할 수 있습니다:

```javascript
import { createApi } from '@reduxjs/toolkit/query';

/* React에 특화된 엔트리 포인트입니다.
   정의된 엔드포인트에 따라 자동으로 Hooks를 생성합니다. */
import { createApi } from '@reduxjs/toolkit/query/react';
```

평소 React를 사용하던 것처럼 `createApi`를 import하는 것으로 시작합니다. 그리고 다음과 같이 서버의 base URL과 상호작용하고자 하는 엔드포인트를 나열하는 API slice를 정의합니다:

```typescript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import { Pokemon } from './types';

// base URL과 사용하기로 예상되는 엔드포인트들을 정의합니다.
export const pokemonApi = createApi({
  reducerPath: 'pokemonApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://pokeapi.co/api/v2/' }),
  endpoints: (builder) => ({
    getPokemonByName: builder.query<Pokemon, string>({
      query: (name) => `pokemon/${name}`,
    }),
  }),
});

// 함수형 컴포넌트에서 사용할 수 있도록 Hooks를 내보냅니다.
// 정의된 엔드포인트들을 기반으로 자동으로 생성됩니다.
export const { useGetPokemonByNameQuery } = pokemonApi;
```

### 저장소 구성하기

API slice는 Redux slice 리듀서와 구독 수명을 관리하는 커스텀 미들웨어를 자동으로 생성하고, 이를 포함하고 있습니다. 다음과 같이 둘 모두 Redux 저장소에 등록해야 합니다:

```typescript
import { configureStore } from '@reduxjs/toolkit';
// React를 사용 중일 경우, '@reduxjs/toolkit/query/react'로부터 import해도 됩니다.
import { setupListeners } from '@reduxjs/toolkit/query';
import { pokemonApi } from './services/pokemon';

export const store = configureStore({
  reducer: {
    // 자동으로 생성된 Redux slice 리듀서
    [pokemonApi.reducerPath]: pokemonApi.reducer,
  },
  // RTK Query의 캐싱, 캐시 무효화, 폴링 등을 포함한
  // 여러 유용한 기능들을 활성화하는 api 미들웨어
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(pokemonApi.middleware),
});

// setupListeners()는 선택 사항이지만, refetchOnFocus/refetchOnReconnect를 위해서는 필수적으로 사용해야 합니다.
// 자세한 내용은 `setupListeners` 문서를 참조하세요.
setupListeners(store.dispatch);
```

### 컴포넌트에서 Hooks 사용하기

마지막으로, API slice로부터 자동 생성된 React Hooks를 컴포넌트 파일에 import하고 필요한 파라미터와 함께 Hooks를 호출하는 방식으로 사용하면 됩니다:

```javascript
import { useGetPokemonByNameQuery } from './services/pokemon'

export default function App() {
  // Hooks를 사용하면 자동으로 데이터를 가져오고 쿼리로부터 얻은 값을 반환합니다.
  const { data, error, isLoading } = useGetPokemonByNameQuery('bulbasaur');
  // 각각의 Hooks는 생성된 엔드포인트 아래에서도 접근 가능합니다:
  // const { data, error, isLoading } = pokemonApi.endpoints.getPokemonByName.useQuery('bulbasaur');

  // 데이터와 로딩 상태를 기준으로 UI를 렌더링합니다.
}
```

RTK Query는 자동으로 컴포넌트 마운트 과정에서 데이터를 가져오고, 파라미터가 바뀔 때마다 데이터를 다시 가져오고, 결과로 `{data, isFetching}` 값을 제공하며, 이 값들이 바뀔 때마다 컴포넌트를 리렌더링합니다.

