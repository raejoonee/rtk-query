---
description: RTK Query의 환경을 설정하고 데이터 가져오기 기능을 사용하는 방법을 안내합니다.
---

# RTK Query 빠르게 시작하기

_이 문서를 완벽하게 이해하려면_ [_Redux에서 사용하는 용어와 Redux의 기본 개념_](https://redux.js.org/tutorials/fundamentals/part-2-concepts-data-flow)_을 알고 있어야 합니다._

## 소개

RTK Query 빠르게 시작하기에 오신 것을 환영합니다! **이 문서는 Redux Toolkit의 RTK Query를 사용한 데이터 가져오기 기능을 간략히 소개하고 이 기능을 어떻게 올바르게 사용할 수 있는지를 가르쳐 줄 것입니다.**

[RTK Query 소개](rtk-query.md) 문서에서 설명했던 대로, **RTK Query**는 데이터 가져오기\(fetching\)와 캐싱하기\(caching\)를 위한 강력한 도구입니다. **데이터를 가져오고 캐싱하는 로직을 직접 일일히 번거롭게 작성하는 과정을 없애고,** 웹 애플리케이션이 데이터를 불러오는 보편적인 과정을 단순화하기 위해 탄생했습니다.

RTK Query는 **Redux Toolkit 패키지에 포함된 선택적 애드온**이며, Redux Toolkit에 있는 다른 API들의 위에 기능이 구현되어 있으므로 함께 사용할 수 있습니다.

### 이 문서를 읽기 전에

이 문서에서는 여러분이 Redux Toolkit을 React와 함께 사용하고 있다고 가정하고 있습니다. 하지만 다른 환경에서도 RTK Query를 사용할 수 있습니다. 마찬가지로 모든 예제는 모든 애플리케이션 코드를 `src` 폴더 내에 두는 [전형적인 Create-React-App 폴더 구조](https://create-react-app.dev/docs/folder-structure/)를 따르고 있지만, 여러분이 원하는 다른 패턴이 있다면 그에 맞게 조정할 수도 있습니다.

## 저장소와 API 서비스 설정하기

RTK Query가 어떤 방식으로 동작하는지 보려면 간단한 사용 예제를 만들어 보면 좋습니다. 이번 예제에서는 RTK Query의 자동 생성된 React Hook을 사용하도록 하겠습니다.

### API 서비스 만들기

먼저, 공용으로 사용 가능한 [PokeAPI](https://pokeapi.co/)_\(역주: 포켓몬 데이터를 제공하는 API입니다.\)_를 처리하는 서비스를 만들어 보겠습니다.

```typescript
// React를 사용 중인 경우 React를 위한 엔트리 포인트에서 import합니다.
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

RTK Query는 모든 API에 대한 정보를 한 곳에 정의하는 방식을 사용합니다. 이 방식은 `react-query`나 `swr`과 같은 다른 라이브러리에서 사용하는 방식과 큰 차이를 보입니다. 이런 방식을 사용하게 된 이유는 여러 가지가 있습니다. 가장 큰 이유는 애플리케이션 전체에 걸쳐서 서로 다른 파일 여럿에 수많은 커스텀 Hook을 두는 방식보다 중심 장소 한 곳에 모든 API 정의를 두는 방식이 API 요청 방식, 캐시 무효화, 애플리케이션 설정 등의 정보를 추적하는 데 훨씬 용이하다고 생각했기 때문입니다.

보통 애플리케이션이 사용하고자 하는 base URL 하나 당 API slice 하나를 두는 편입니다. 예를 들어 `/api/posts`와 `/api/users` 둘로부터 데이터를 가져오고자 한다면, `/api/`를 base URL로 두는 API slice 하나만 작성하면 됩니다. 그리고 이 API slice의 `endpoints`에 `posts`와 `users`를 정의합니다. 이런 방식을 사용하면 정의된 엔드포인트들 사이 정의된 [`tag`](https://redux-toolkit.js.org/rtk-query/usage/automated-refetching#tags)를 통해 [자동으로 데이터 다시 가져오기\(automated re-fetching\)](https://redux-toolkit.js.org/rtk-query/usage/automated-refetching)를 효율적으로 수행할 수 있습니다.

이렇게 하나의 API slice에 모든 엔드포인트들을 포함하는 형태를 유지하여 효율을 챙기면서도, 유지보수성을 위해 엔드포인트 정의만큼은 여러 파일로 나누고 싶을 수도 있습니다. 그런 당신을 위해 준비했습니다. [코드 스플리팅](https://redux-toolkit.js.org/rtk-query/usage/code-splitting) 문서에서 설명하는 `injectEndpoints` 프로퍼티를 사용해 다른 파일 여럿으로 흩어진 API 엔드포인트 정의를 하나의 API slice 정의로 합치는 방법을 알아보세요!

### 저장소에 API 서비스 추가하기

RTK Query는 Redux 루트 저장소에 포함되어야 하는 slice 리듀서와 데이터 가져오기를 처리하는 커스텀 미들웨어를 자동으로 생성합니다. 이 둘은 모두 Redux 저장소에 무조건 추가되어야 합니다.

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

### `Provider`로 애플리케이션 감싸기

Redux 저장소를 구성할 때와 마찬가지로, 저장소를 `Provider`로 감싸줍니다.

```typescript
import * as React from 'react';
import { render } from 'react-dom';
import { Provider } from 'react-redux';

import App from './App';
import store from './app/store';

const rootElement = document.getElementById('root');
render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
);
```

## 컴포넌트에서 쿼리 사용하기

### 기본 예제

서비스를 정의한 뒤, 요청을 만들기 위해 \(자동으로 생성된\) Hooks를 import하여 사용할 수 있습니다.

```typescript
import { useGetPokemonByNameQuery } from './services/pokemon'

export default function App() {
  // Hooks를 사용하면 자동으로 데이터를 가져오고 쿼리로부터 얻은 값을 반환합니다.
  const { data, error, isLoading } = useGetPokemonByNameQuery('bulbasaur');
  // 각각의 Hooks는 생성된 엔드포인트 아래에서도 접근 가능합니다:
  // const { data, error, isLoading } = pokemonApi.endpoints.getPokemonByName.useQuery('bulbasaur');

  return (
    <div className="App">
      {error ? (
        <>앗, 오류가 발생했습니다.</>
      ) : isLoading ? (
        <>로딩중...</>
      ) : data ? (
        <>
          <h3>{data.species.name}</h3>
          <img src={data.sprites.front_shiny} alt={data.species.name} />
        </>
      ) : null}
    </div>
  )
}
```

요청을 만들고 난 뒤, 다양한 방법을 통해 해당 요청에 대한 상태를 추적할 수 있습니다. `data`, `status`, `error` 값을 확인해서 어떤 UI를 렌더링할 지 결정할 수 있습니다. 추가로, `useQuery` Hook은 가장 최근 요청에 대해 `isLoading`, `isFetching`, `isSuccess`, `isError` 등과 같은 유용한 boolean 값을 제공합니다. 그런데, 만약 여러 컴포넌트에서 동일한 포켓몬 데이터에 대한 정보를 불러올 때는 무슨 일이 일어날까요?

### 심화 예제

RTK Query는 같은 쿼리를 구독하고 있는 모든 컴포넌트에 대해 항상 같은 데이터를 사용하도록 합니다. 중복된 요청은 자동으로 제거하기 때문에 개발자가 퍼포먼스 최적화를 위해서 in-flight request\(이미 요청이 시작됐지만 아직 완료되지 못한 요청\)가 있는지 일일히 직접 확인할 필요가 없습니다. 아래 코드를 실행하고 브라우저 관리자 도구의 네트워크 패널을 관찰해보세요.

{% embed url="https://codesandbox.io/embed/github/reduxjs/redux-toolkit/tree/master/examples/query/react/advanced?autoresize=1&fontsize=12&hidenavigation=1&theme=dark" %}

컴포넌트 4개를 렌더링하지만, 네트워크 요청은 3번만 발생하는 것을 확인할 수 있습니다. `bulbasaur`_\(역주: 이상해씨\)_의 데이터를 요청하는 컴포넌트가 두 개 있지만, 요청은 한 번만 발생하기 때문입니다. 또한, 두 컴포넌트의 로딩 상태가 자동으로 동기화되는 것도 확인할 수 있습니다. 드롭다운 값을 바꿔가면서 이 방식이 어떻게 계속되는지 한 번 확인해 보세요.

