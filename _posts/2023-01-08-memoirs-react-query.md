---
title: 회고록 - React Query를 도입하며
author: Roen
date: 2023-01-08 17:22:00 +0800
categories: [회고록]
tags: [회고록, React, React Native, React Query]
pin: false
math: true
mermaid: true
---

{: .prompt-tip }

{: .prompt-tip }

> 이 글은 프로젝트를 진행하며 겪었던 문제를 어떻게 개선했는지 되돌아보고, 더 나은 프로젝트를 진행하기 위한 회고 글입니다.

## 개요

프로젝트에 들어가기에 앞서, 업무의 효율성을 위해 어떠한 프레임워크나 라이브러리를 사용할지 정하는 것은 중요하다.
<br/>사내에서는 주로 React Native와 React를 사용하고 있었고, 이러한 라이브러리로 개발을 진행하다 보면 규모가 커짐에 따라 컴포넌트가 복잡해지면서 관리하는 상태도 당연히 많아지게 된다.
<br/> 여기서 상태는 일종의 데이터이고, 데이터가 변경된다는 것은 상태가 변경된다는 것이다.
<br/>상태가 변경됨으로써, 사용자에게 보여주는 화면의 랜더링에 영향을 주게 되기 때문에 효율적으로 상태를 관리하는 것이 중요하다.
<br/>이러한 이유로 클라이언트에서 상태를 어떻게 관리할 것인가에 대해 팀원들과 논의하고 고민하는 시간을 가지게 되었다.

## 도입 계기

결론부터 말하자면 신규 프로젝트에 [React Query](https://tanstack.com/query/latest)[^React-Query]를 도입하게 되었다.
<br/>
그래서 왜 `React Query`를 도입하게 되었을까?
<br/>여러 이유가 있지만, 가장 개선되어야 한다고 생각했던 문제들을 `React Query`를 도입함으로써, 해결할 수 있었기 때문이다. 그 이유를 추려보자면 아래와 같다.

### 1. 데이터의 동기화

기존 서비스를 유지 보수하며 겪었던 경험을 토대로 이야기하고자 한다.
<br/>
가령 커뮤니티 상세 화면에서 게시글에 대해 `좋아요`를 클릭할 수 있고, 커뮤니티 리스트를 보여주는 화면에서는 해당 게시글이 `좋아요`를 받은 갯수를 표시하고 있다고 치자.
<br/>사용자가 해당 게시글에 대해 `좋아요`를 클릭하고, 다시 커뮤니티 리스트를 보여주는 화면으로 되돌아갔을 때, 화면이 갱신되어 있지 않다면 어떨까?
<br/>분명 사용자는 `좋아요`를 한 글인데 리스트에서는 `좋아요`를 받은 갯수가 변하지 않는다는 것을 확인한다면, 사용자 입장에서는 버그라고 생각할 것이다.
<br/>
즉 분명히 같은 데이터인데 상세 화면은 최신 데이터로 업데이트되어 보여지고, 리스트에서는 예전 데이터가 보여지고 있어 데이터가 일치하지 않는 문제가 발생하게 된다. 이러한 문제는 저하된 사용자 경험을 제공해준다.

### 2. 너무 긴 보일러 플레이트(Boiler Plate) 코드[^Boiler-Plate]

기존에는 Redux와 Redux-saga를 사용해 비동기 처리 같은 사이드 이펙트를 처리하고 있었다.
<br/>
Redux-saga는 사용 빈도가 낮은 생소한 제너레이터 함수를 사용하고 있어 이에 대한 러닝 커브가 높았고, 하나의 기능을 구현하기 위해 긴 보일러 플레이트(Boiler Plate) 코드를 작성해야 했다.

### 3. 방대한 양의 서버 상태

클라이언트 단에서만 관리되는 데이터보다 서버와의 통신을 통해 가져오는 데이터가 월등히 많았지만, 데이터의 상태를 관리하는 로직은 통합되어 있었다.
<br/>이러한 로직을 분리하면 각각의 기능에 더 집중할 수 있게 되고, 효율적으로 상태 관리를 할 수 있을 거라고 생각했다.

<br/>
프론트에서 효율적으로 상태를 관리할 수 있는 라이브러리가 계속해서 생겨나고 발전하는 상황에서 기존에 사용하고 있던 방법을 고집할 필요성을 느끼지 못했고,
위의 3가지 문제를 쉽게 개선할 수 있도록 도와주는 `React Query`를 도입했다.

## 개선된 점

### 1. 상태 관리 개선

Redux의 리듀서는 순수함수여야 한다는 이유로 사이드 이펙트를 처리하기 위해 Redux-saga라는 미들웨어를 사용해 비동기 로직를 처리했었다.
<br/>
React Query를 사용하면 직접 구현하지 않아도 훨씬 수월하게 비동기 사이클을 관리할 수 있었고, 기존 방식과 비교하여 보일러 플레이트 코드를 대략 30% 제거할 수 있었다.
<br/>
또한 Redux에서는 전역적으로 클라이언트에서 관리되는 상태만 남겨두고, 최대한 서버 상태를 관리하는 로직을 분리함으로써, 코드 추적이 용이해졌다.

<span style="font-size:22px"><b>AS-IS</b></span>

- <span style="font-size:20px">Reudx-Saga 미들웨어로 비동기 로직 처리</span>

```ts
// modules/products.tsx
import { ActionType, createAsyncAction, createReducer } from "typesafe-actions";
import { Product } from "../types/products";
import { call, put } from "redux-saga/effects";
import { productsApi } from "../api/products";
import { AxiosError } from "axios";

export const PRODUCTS_REQUEST = "FETCH_PRODUCTS_REQUEST";
export const PRODUCTS_SUCCESS = "FETCH_PRODUCTS_SUCCESS";
export const PRODUCTS_FAILURE = "FETCH_PRODUCTS_FAILURE";

export const getProductsAll = () => ({ type: PRODUCTS_REQUEST });

export type ProductsState = {
  productList: Product[];
  isLoading: boolean;
  isError: boolean;
};

// 액션 생성
export const fetchProductsAsync = createAsyncAction(
  PRODUCTS_REQUEST,
  PRODUCTS_SUCCESS,
  PRODUCTS_FAILURE
)<null, Product[], AxiosError>();

// 비동기 로직 처리를 위한 사가 작성
export function* loadProductSaga(): Generator {
  try {
    const res = yield call(productsApi.getAll);
    yield put(fetchProductsAsync.success(res as unknown as Product[]));
  } catch (error) {
    yield put(fetchProductsAsync.failure(error as AxiosError));
  }
}

const actions = { fetchProductsAsync };
export type ProductsAction = ActionType<typeof actions>;

const initialState: ProductsState = {
  productList: [],
  isLoading: false,
  isError: false
};

// 리듀서 생성
export const products = createReducer<ProductsState, ProductsAction>(
  initialState,
  {
    [PRODUCTS_REQUEST]: (state) => ({ ...state, isLoading: true }),
    [PRODUCTS_SUCCESS]: (state, action) => ({
      ...state,
      isLoading: false,
      productList: action.payload
    }),
    [PRODUCTS_FAILURE]: (state) => ({
      ...state,
      isLoading: false,
      isError: true
    })
  }
);
```

<br/>
- <span style="font-size:20px">실제 컴포넌트에서 사용</span>

```tsx
// ProductsPage.tsx
function ProductsPage() {
  const dispatch = useDispatch();
  const { productList, isError, isLoading } = useSelector(
    (state: RootState) => state.products
  );

  useEffect(() => {
    // dispatch 함수로 리듀서 실행
    dispatch(getProductsAll());
  }, []);

  if (isLoading && !productList.length) {
    return (
      <div>
        <p>로딩중...</p>
      </div>
    );
  }

  if (isError) {
    return (
      <div>
        <p>에러 발생</p>
      </div>
    );
  }

  return (
    <div>
      Products
      <ProductList productList={productList} />
    </div>
  );
}
```

{: .prompt-info }

> <span style="color:red">(2023.11 추가 완료)</span> [참고 코드 링크](https://github.com/Roen77/react-example/blob/master/src/modules/products.ts)

{: .prompt-info }

<br/>

<span style="font-size:22px"><b>TO-BE</b></span>

- <span style="font-size:20px">React Query로 비동기 로직 처리</span>

```tsx
// ./constans.ts
export const productKeys = {
  getAll: ["products"]
};

// ProductsPage.tsx
function ProductsPage() {
  // const dispatch = useDispatch();
  // const { productList, isError, isLoading } = useSelector(
  //   (state: RootState) => state.products
  // );

  // useEffect(() => {
  //   dispatch(getProductsAll());
  // }, []);

  const {
    isLoading,
    data: productList,
    isError
  } = useQuery(productKeys.getAll, productsApi.getAll);

  if (isLoading && !productList) {
    return (
      <div>
        <p>로딩중...</p>
      </div>
    );
  }

  if (isError) {
    return (
      <div>
        <p>에러 발생</p>
      </div>
    );
  }

  return (
    <div>
      Products
      <ProductList productList={productList} />
    </div>
  );
}
```

{: .prompt-info}

> <span style="color:red">(2023.11 추가 완료)</span> [참고 코드 링크](https://github.com/Roen77/react-example/blob/master/src/pages/ProductsQueryPage.tsx)

{: .prompt-info }

<br/>
위의 예제는 실제 사용된 데이터가 아닌 [상품을 가져오는 오픈 API](https://fakestoreapi.com/)를 사용해서 상품 리스트를 가져오는 상황을 간단하게 구현해본 것이다.
<br/>
Reudx-Saga로 비동기 처리 로직의 로딩과 에러 처리를 직접 구현했다면, React Query에서는 쉽게 비동기 처리 사이클을 파악하고 핸들링이 가능하다.

### 2. 빠른 피드백으로 사용자 경험 개선

- <b style="font-size:18px">2-1. 사용자의 인터렉션에 빠르게 대응하여 피드백을 제공하고, 언제나 사용자에게 데이터가 업데이트된 최신의 상태를 보여줌으로써 데이터의 동기화가 가능해졌다.</b>

실제로 겪었던 경험을 토대를 기반으로, React Native에서 최신 데이터를 사용자에게 어떻게 제공해주었는지 예시를 들어 설명하고자 한다.
<br/>

> 상품 리스트를 보여주는 A화면과 상품 리스트를 선택해서 들어가는 상품 상세를 보여주는 B화면이 있다고 치자.
> <br/>사용자가 B화면에서 상품의 제목을 수정한다음, 다시 뒤로가기로 A화면으로 돌아간다고 가정한다면 당연히 A화면에서도 내가 수정한 제목으로 변경된 데이터를 가지고 있는 상품 리스트가 보여야 한다.

 <br/>
React Native에서는 웹 브라우저가 라우터 상태를 처리하는 방식과 개념적으로 유사한 방식으로 앱을 탐색하는데, 사용자가 상호 작용할 때 앱이 내비게이션 스택에서 항목을 푸시하고 팝업하므로 사용자에게 다른 화면이 표시된다.
<br/>즉, 스택이 쌓이는 구조이기 때문에 A화면이 마운트되고 A화면에서 B화면으로 이동할 경우, A화면은 스택에 마운트된 상태로 유지된다. 이상태로 B에서 다시 A로 뒤로가기를 할 경우, 이미 마운트된 상태인 A화면에서는 마운트될 때 호출되는 useEffetct 훅이나 componentDidMount 라이프 사이클이 다시 실행되지 않는다.<br/> B화면에서 상품의 정보를 변경한다면, A화면에서 상품 리스트를 가져오는 API를 다시 호출해서 상태를 업데이트하여 최신 데이터로 만들 수 있을 것 같은데 이러한 상황이라면 어떻게 해야 할까?
<br/>뒤로 가기 시에 `focus` 이벤트를 발생시키거나 스택을 초기화하는 등의 여러가지 방법으로, A화면에서 마운트 훅을 실행시켜서 다시 상품 리스트 API를 호출하여 데이터를 갱신시킬 수 있을 것이다.
<br/>React Query는 이러한 과정을 간소화하고, 자연스럽게 데이터를 업데이트 할 수 있는 방법을 제공해주고 있다.
<br/>
<br/>

```tsx
// components.tsx

import { useMutation, useQueryClient } from "@tanstack/react-query";

const queryClient = useQueryClient();

// update
const updateTitleMutation = useMutation(productsApi.updateProductTitle, {
  onSuccess() {
    // update한 후 상품 리스트 데이터에 해당하는 쿼리 키를 무효화시켜 데이터를 업데이트 시킬 수 있다.
    queryClient.invalidateQueries(productKeys.getAll);
  }
});

// 업데이트 실행 함수
const updateTitle = () => {
  updateTitleMutation.mutate({ id, title: text.trim() });
};
```

<br/>
이러한 방법으로 굳이 스택을 초기화하거나 별도의 이벤트 핸들링을 추가하지 않고도 B 화면에서 데이터를 수정한 후, A 화면에 보이고 있는 상품 리스트를 바로 갱신시킴으로써, 사용자 경험을 향상시킬 수 있다.
<br/>
위는 단지 하나의 예일뿐 거창하게 설명했지만, 사실 동일한 데이터라면 어디서나 같은 내용의 일정한 데이터를 보여주어야 하는 것은 당연한 일이다.
<br/>
<br/>

- <b style="font-size:18px">2-2. 특정 상태의 변경은 Optimistic Update로 딜레이 없는 UI를 제공하게 되었다.</b>

중고 장터 관련 커뮤니티를 개발하는 과정에서, 사용자가 상품의 판매 상태를 변경함에 따라 UI를 일부 변경함으로써 사용자에게 변경 상태를 바로바로 알려주어야 했다.
<br/>상품의 상태가 변경됨에 따라 변경 상태를 갱신하는 API를 호출하고 있었기에, API 응답을 받은 후 UI를 갱신시켰을 때 약간의 버벅거리는 현상이 간헐적으로 보였다.
<br/>물론 빠른 네트워크의 상황에서는 문제가 없을진 모르지만, 사용자 입장에서는 어떠한 행동을 했을 경우 그에 대한 응답이 지연된다면 반응이 제대로 되었는지 파악하기 힘들것이다.
<br/> 이러한 점을 개선하기 위해 [Optimistic Update](https://tanstack.com/query/v4/docs/react/guides/optimistic-updates)[^Optimistic-Update]를 사용하여 자연스러운 UI를 제공하게 되었다.

### 3. 네트워크 비용 절감

정적 데이터와 자주 변경되는 데이터를 구분하여 서버 상태를 관리할 수 있게 되었다.
<br/>
React Query에서는 staleTime 과 cacheTime를 적절히 조절해서 서버의 상태를 관리할 수 있다.
<br/> [React Query 블로그](https://tkdodo.eu/blog/practical-react-query)에서 발췌한 내용을 번역하면, staleTime 과 cacheTime는 아래와 같다.

{: .prompt-info }

{: .prompt-info }

> <b>staleTime</b>: 쿼리가 새로운 쿼리에서 오래된 쿼리로 전환될 때까지의 기간입니다.
> <br/> 쿼리가 최신인 한 데이터는 항상 캐시에서만 읽혀지며 네트워크 요청은 발생하지 않습니다. 쿼리가 오래된 경우(기본값은 즉시) 캐시에서 데이터를 계속 가져오지만 특정 조건에서는 백그라운드 다시 가져오기가 발생할 수 있습니다.

> {: .prompt-info }

{: .prompt-info }

> <b>cacheTime</b>: 비활성 쿼리가 캐시에서 제거될 때까지의 기간입니다. 기본값은 5분입니다.
> <br/>등록된 관찰자가 없는 즉시 쿼리가 비활성 상태로 전환됩니다. 즉, 해당 쿼리를 사용하는 모든 구성 요소가 마운트 해제된 경우입니다.

<br/>
서울,경기 같은 시군구 지명 데이터는 쉽게 변하는 데이터가 아니기 때문에 이러한 정적인 데이터는 staleTime을 Infinity로 설정하여 쿼리를 fresh한 상태로 유지시켜 불필요한 네트워크 요청을 줄였다.
<br/>자주 변경되는 데이터는 staleTime과 cacheTime를 상황에 따라 조율하여 효과적으로 서버 상태를 관리할 수 있게 되었다.

### 4. 개발자 경험 개선

라이브러리를 사용하는 이유가 여러 가지 있겠지만, 가장 큰 이유는 라이브러리에서 제공해 준 기능을 사용함으로써 개발 시간을 단축시키고, 효율성을 증대시킬 수 있다는 장점이 있기 때문이다.
<br/>
React Native으로 앱 서비스를 개발할 때, 앱의 특성상 주로 손가락 제스처에 의해 상호작용이 이루어지기 때문에 무한 스크롤 방식이 많이 사용된다.
<br/>React Query는 무한 스크롤 기능에 많은 기능을 제공해 주기 때문에 이러한 기능을 충분히 활용하여 기능 개발이 수월했다.

## 아쉬운 점

React Query를 도입하며, 개선이 필요하다고 생각된 문제들을 해결할 수 있게 되어 단점보다 장점이 훨씬 컸지만 그래도 아쉬운 부분은 있었다.
<br/>React Query는 공식 문서에서 설명하듯이 서버 상태의 관리에 중점을 두었기에, 클라이언트 단에서만 관리되는 전역 상태를 관리하는데 쓰기에는
불편했다.<br/>그렇기 때문에 Redux를 완전히 걷어내지는 않았고, 대신 기존 Reudx를 보완한 [Redux-toolkit](https://redux-toolkit.js.org/)를 사용하여 전역 상태 관리를 하여 기존에 반복되는 코드가 많아서 복잡해졌던 부분을 개선했다.

## 정리

정리하자면 React 같은 라이브러리를 사용하여 개발을 진행하다 보면, 규모와 관리하는 상태도 복잡해지고 많아진다.
<br/>그렇기 때문에 렌더링에 영향을 끼치는 상태를 잘 관리해야 코드 추적도 용이해지고, 유지 보수도 수월하기 때문에 이러한 상태 관리는 중요하다.
<br/>서버 상태를 관리해야 하는 상황이 월등히 많은 프로젝트에 서버 상태를 관리에 특화된 React Query를 사용함으로써, 수 많은 비동기 로직을 처리하는 과정에서 많은 도움이 되었다.
<br/>상태를 관리하는데 도움을 주는 여러 라이브러리나 방식이 존재하기 때문에 자신이 개발하는 환경이나 프로젝트에 맞춰 어떤 방식을 사용할 것인지는 신중히 고려해보아야 한다.

{: .prompt-info }

> <span style="color:red">(2023.11 추가 완료)</span> [참고 코드 링크](https://github.com/Roen77/react-example)

{: .prompt-info }

> <span style="color:red">(2023.11 추가 완료)</span> [사이트 링크](https://dynamic-kangaroo-e65a7d.netlify.app/)

{: .prompt-info }

<br/>
<span style="background-color:#E3F2FD; padding:4px; border-radius:2px; font-size:14px;">주석</span>

[^React-Query]: 서버 상태를 효율적으로 관리하도록 도와주는 라이브러리
[^Boiler-Plate]: 프로그래밍에서 자주 반복되는 작업이나 패턴을 가진 코드
[^Optimistic-Update]: 서버로 보낸 요청이 정상적일 것을 예상하고, response가 오기 전 클라이언트의 데이터를 미리 변경시키는 작업
