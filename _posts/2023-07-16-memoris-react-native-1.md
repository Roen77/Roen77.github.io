---
title: 회고록- React Native 오류 개선과 성능 최적화
author: Roen
date: 2023-07-16 17:22:00 +0800
categories: [회고록]
tags: [회고록, React Native]
pin: false
math: true
mermaid: true
---

{: .prompt-tip }

{: .prompt-tip }

> 이 글은 프로젝트를 진행하며 겪었던 문제를 어떻게 개선했는지 되돌아보고, 더 나은 프로젝트를 진행하기 위한 회고글입니다.

## 개요

React Native로 앱을 개발하면서 자주 접했던 오류를 어떻게 개선하고, 성능을 어떻게 최적화했는지 정리해보려 한다.

## 1. Please report: Excessive number of pending callbacks: 501. Some pending callbacks that might have leaked by never being called from native code

### 문제

React Native로 개발을 하는 도중, 에러는 아니었지만 간헐적 경고메세지로 `Please report: Excessive number of pending callbacks: 501. Some pending callbacks that might have leaked by never being called from native code` 가 떴다.
<br/> 해석해보자면 보류중인 콜백의 수가 너무 과도하다는 뜻으로, 네이티브 코드에서 호출되지 않아 누출될 수 있는 대기중인 콜백이 많이 있다는 뜻이다.
<Br/>단순히 경고로 치부하고 넘어가기엔 어마 무시한 오류메시지였고 꼭 해결할 필요가 있었다.

### 개선

[react-native issue](https://github.com/facebook/react-native/issues/27483)를 찾아보니 해당 메세지가 뜨는 이유는 다양했지만,
결론적으로 말하자면 React Navigation 라이브러리에서 제공해주는 훅을 잘못 사용한 이유였다.
<br/>
React Navigation 라이브러리에서는 화면에 초점이 맞춰졌을 때, 사이드 이펙트를 실행할 수 있도록 [useFocusEffect](https://reactnavigation.org/docs/use-focus-effect/) 훅을 제공해주고 있다.
<Br/>즉, 화면이 focus 되었을 때 해당 훅이 실행되는데 useEffect 훅과 비슷하게 return으로 cleanup 함수를 실행시켜 뒷정리를 할 수 있듯이 해당 훅에서도 뒷정리를 할 수 있는 기능을 제공해 준다.
<br/>
`useFocusEffect`에서는 화면이 blur 되었을 때, return으로 해당 effect를 cleanup 할 수 있다.
<br/>여기서 문제는 화면이 blur 되었을 때 아무런 처리도 하지 않아 계속해서 화면이 focus 되었을 때 많은 콜백들이 쌓인 게 문제가 되었다.

```tsx
useFocusEffect(
  useCallback(() => {
    let isActive = true;

    const fetchProducts = async () => {
      try {
        const products = await productApi.getItem(id);

        if (isActive) {
          setData(products);
        }
      } catch (e) {
        //error
      }
    };

    fetchProducts();

    return () => {
      isActive = false;
    };
  }, [id])
);
```

서버에서 데이터를 가져오는 등의 비동기 효과를 실행하는 경우, `cleanup` 함수로 정리 기능에서 요청을 취소하는지 확인하는 것이 중요하고,
만약 취소 메커니즘을 제공하지 않는 API를 사용하는 경우 위처럼 상태를 업데이트 하지 않도록 하는 것이 중요하다.

`useCallback` 으로 감싸 effect가 자주 호출되는 것을 방지하고 `cleanup` 함수로 뒷정리를 해주어 해당 경고 메세지를 제거할 수 있었다.

## 2. 리스트 성능 이슈

### 문제

사용자에게 많은 데이터를 보여주는 방법은 여러 가지가 있지만, 앱에서는 페이지 네이션보다는 무한 스크롤 기능으로 스크롤 시 자연스럽게 데이터를 제공해 주는 방식을 훨씬 많이 사용할 것이다.
<br/>
사용자가 스크롤을 계속해서 하다 보면 그만큼 보여줄 데이터도 계속해서 늘어나기 때문에 리스트를 렌더링 하는 데이터의 양도 많아지게 된다.
<br/>

> 10개씩 리스트를 보여주고, 스크롤 시 다음 10개의 리스트를 추가적으로 보여준다고 했을 때, 계속해서 랜더링하는 데이터의 양은 누적된다. 10개,20개,30개,40개..

<br/>
이렇게 많은 리스트를 랜더링하기에 최적화된 기능을 다행히 React Native에서 제공해주고 있다.
<Br/> 바로 [FlatList](https://reactnative.dev/docs/flatlist)로, 화면에 적절한 양의 데이터만 렌더링한 뒤, 사용자의 제스쳐에 의해 스크롤이 움직일 때 필요한 부분을 추가적으로 랜더링함으로써 퍼포먼스 향상을 기대할 수 있다.
<br/>
하지만 단순히 `FlatList`를 사용하는 것에 그치지 않고, `FlatList` 를 어떻게 구성해야 성능 최적화를 더 끌어올릴 수 있는지 정리해보려 한다.

- <b>ProductList.tsx</b>

```tsx
function ProductList() {
  const { isLoading, data, isFetchingNextPage, fetchNextPage, hasNextPage } =
    useInfiniteQuery(productKeys.getAll, productsApi.getAll, {
      getNextPageParam: (lastPage, pages) => {
        if (lastPage && lastPage.length === 0) {
          return;
        }
        return pages.length + 1;
      }
    });

  const productList = useMemo(() => (data?.pages || []).flat(), [data?.pages]);

  const renderItem: ListRenderItem<Product> = useCallback(
    ({ item }) => <ProductItem item={item} />,
    []
  );

  const onEndReached = () => {
    if (isFetchingNextPage) {
      return;
    }
    hasNextPage && fetchNextPage();
  };

  if (isLoading) {
    return null;
  }

  return (
    <FlatList
      onEndReached={onEndReached}
      keyExtractor={(_, index) => index.toString()}
      data={productList || []}
      renderItem={renderItem}
    />
  );
}

export default ProductList;
```

해당 예제에서는 한번에 10개씩 상품의 리스트를 가져오고 있고, 스크롤이 화면 끝에 닿았을 때 다음 10개의 데이터를 불러와서 화면에 랜더링해주고 있다.

<br/>
- <b>ProductItem.tsx</b>

```tsx
import { Image, Pressable, StyleSheet, Text, View } from "react-native";
import { Product } from "../types/products";
import React = require("react");
import { NavigationProp, useNavigation } from "@react-navigation/native";
import { RootStackParamList } from "../types/navigation";
type Props = {
  item: Product;
};

function ProductItem({ item }: Props) {
  console.log("item:", `${item.id}`);
  const navigation = useNavigation<NavigationProp<RootStackParamList>>();
  const navigateDetail = () => {
    navigation.navigate("Detail");
  };
  return (
    <Pressable
      style={styles.itemContainer}
      onPress={navigateDetail}
      key={item.id}
    >
      <Text style={styles.title}>{item.title}</Text>
      <View style={styles.imageContainer}>
        <Image
          style={{ width: 50, height: 50, resizeMode: "contain" }}
          source={{ uri: item.image }}
        />
      </View>
      <Text>${item.price}</Text>
      <Text>{item.description}</Text>
    </Pressable>
  );
}

export default ProductItem;

const styles = StyleSheet.create({
  itemContainer: {
    paddingVertical: 30,
    paddingHorizontal: 5,
    borderWidth: 1,
    marginVertical: 10,
    marginHorizontal: 5
  },
  imageContainer: {
    paddingVertical: 10,
    justifyContent: "center",
    alignItems: "center"
  },
  title: {
    fontSize: 20,
    fontWeight: "700"
  }
});
```

<br/> `ProductItem` 컴포넌트는 리스트의 각각의 아이템을 랜더링해주고 있다.

<br/>
ProductList 컴포넌트에서 처음 10개의 데이터를 불러왔을 때와, 스크롤이 화면 끝에 닿아서 다음 10개의 데이터를 불러올 때 `ProductItem` 컴포넌트가 랜더링이 어떻게 되는지 확인해보자.

- <b>처음 10개의 데이터를 불러왔을 때</b>

  ![Desktop View](https://github.com/Roen77/Roen77.github.io/assets/65719512/1b9b8361-213f-4ae2-bbac-d2e288cdd28f){: width="500" height="589" }

<br/>

- <b>다음 10개의 데이터를 불러왔을 때</b>

  ![Desktop View](https://github.com/Roen77/Roen77.github.io/assets/65719512/81a338b0-597b-4eb5-8c1d-321c82aa569d){: width="500" height="589" }

<Br/>
처음 10개의 데이터를 불러온 이후, 다음 10개의 데이터를 불러와서 리스트의 아이템이 추가되었을때도, 다시 모든 리스트의 아이템이 리랜더링되는 것을 확인 할 수 있다. 불필요하게 리랜더링되는 아이템을 어떻게 하면 최적화할 수 있을까?

### 개선

<Br/> [React Native 공식문서](https://reactnative.dev/docs/next/optimizing-flatlist-configuration)를 보면 힌트를 얻을 수 있다. 공식문서를 참고해서 최적화를 진행해보자.

<b>1. React의 memo와 useCallBack으로 리스트 아이템 메모이제이션</b>

- <b>ProductList.tsx</b>

```tsx
function ProductList() {
  ...
// useCallBack으로 renderItem 함수 메모이제이션
  const renderItem: ListRenderItem<Product> = useCallback(
    ({ item }) => <ProductItem item={item} />,
    []
  );

 ...
}

export default ProductList;
```

<br/>
- <b>ProductItem.tsx</b>

```tsx


function ProductItem({ item }: Props) {
    ....
}

// memo로 props가 변경될때만 리랜더링될 수 있도록 컴포넌트 메모이제이션
export default React.memo(ProductItem);
```

ProductItem 에서는 props인 item이 변경될때만 리랜더링 될 수 있도록 React의 memo로 감싸주고, ProductList 컴포넌트에서는 useCallback으로 감싸주어 랜더링 시 함수가 다시 생성되는 것을 방지했다.
<br/> 그 결과, 리스트 아이템이 추가 될 때, 모든 아이템이 리랜더링 되는 것이 아닌 추가된 아이템만 랜더링 된 것을 확인할 수 있다.

![Desktop View](https://github.com/Roen77/Roen77.github.io/assets/65719512/cf2b0dd5-bcc7-4720-b04e-848108e4f3f1){: width="500" height="589" }

<br/>

<b>2. removeClippedSubviews</b>
<br/>
`removeClippedSubviews` 속성을 `true` 로 주면, 뷰포트 외부에 있는 뷰는 기본 뷰 계층 구조에서 분리되어 기본 스레드에서 소요되는 시간이 줄어들고 기본 렌더링를 할 때 뷰포트 외부의 뷰를 제외하여 프레임이 삭제될 위험이 줄어들어 성능 향상을 기대할 수 있다고 한다.
<br/>
단점도 존재하는데 너무 복잡한 작업을 수행할 경우, 콘텐츠 누락 같은 버그가 발생할 수 있다. 그렇기 때문에 모든 `FlatList`를 사용할 때 해당 속성을 주진 않고, 단순히 리스트 목록을 보여줄 경우에만 해당 속성을 사용했다.

<b>3. windowSize</b>
<br/>
`windowSize`는 기본값이 `21` 이고, 이 숫자의 의미는 해당 기준 위 뷰포트에서 10개의 리스트 아이템, 아래 뷰포트에서 10개의 리스트 아이템, 그 사이에 1개의 리스트 아이템을 의미한다.
<br/> 여기에 전달된 숫자는 한번에 랜더링시켜 보여줄 항목의 수로, `windowSize`의 크기가 클수록 빈공간을 볼 가능성이 줄어들고, 크기가 작을수록
동시에 마운트되는 항목 수가 줄어들어 메모리가 절약될 수 있다.
<br/> 한번에 50개이상의 많은 데이터를 가져오는데 실제 화면에서는 그 중 2개 정도의 데이터만 처음에 보여진다고 하면 `windowSize` 의 숫자를 변경해서 앞뒤에 한번에 마운트되는 항목의 수를 조절할 수 있다.
<br/> 위에서 말했듯이 `FlatList` 퍼포먼스 향상을 위해 `windowSize`를 너무 작게 주었더니, 스크롤을 연속해서 계속 내렸을 경우 빈 화면이 보여지는 현상이 나타났다. 그렇기 떄문에 해당 속성을 무리하게 작게 주기 보다는 한 화면에 보여지는 데이터가 한 두개 정도만 보일 경우에만 `windowSize`를 조율했다.
<br/>

<b>4. 캐시에 최적화된 이미지 사용</b>
<br/>
리스트에서 사용자에게 이미지의 유무를 파악할 수 있도록 미리 썸네일이라는 형태로 이미지를 보여준다.
<br/>이러한 리스트에 있는 이미지는 각각 `new Image()`로 생성된 인스턴스이기 때문에 이미지가 빨리 로드가 되어야 로드된 이후 실행하는 훅이 실행되어 자바스크립트 스레드가 더 빨리 자유로워 진다.
<br/>실제로 대부분의 리스트에서 썸네일 이미지를 사용하고 있었기에 이러한 이미지를 최적화하기 위한 라이브러리를 사용했다.
[react-native-fast-image](https://www.npmjs.com/package/react-native-fast-image) 라이브러리를 사용하여 이미지 캐싱 동작을 적극적으로 활용했고, 리스트 뿐만 아니라 모든 이미지에서 해당 라이브러리를 적용하여 사용했다.

<br/>
실제 업무에서 사용했던 방법 위주로 React Native에서 리스트의 성능 최적화를 하는 방법에 대해 정리해보았다.

## 3. ios 환경에서 스크롤 막대가 가운데로 오는 이슈

### 문제

ios에서 스크롤이 존재하는 A화면에서 다른 화면으로 갔다가 다시 A화면으로 돌아왔을 때, 스크롤이 엇나가는 현상이 간헐적으로 발생했다.
<Br/>찾아보니 이미 react-native issue에 올라와있었고, 오랫동안 활발한 토론을 진행한것으로 보아 자주 발생되는 이슈인 것 같았다.

[ios scrol 막대가 가운데로 이동하는 이슈](https://github.com/facebook/react-native/issues/26610)

### 개선

이슈의 원인은 스크롤이 존재하는 화면에서 scrollIndicatorInsets props의 right 속성이 간헐적으로 먹히지 않아서 발생된 이슈였다.
<br/>
공식 문서를 찾아보니 [scrollIndicatorInset](https://reactnative.dev/docs/scrollview#scrollindicatorinsets-ios)는 `스크롤 뷰 표시기가 스크롤 뷰 가장자리에서 삽입되는 양`으로 기본값은 `{top: 0, left: 0, bottom: 0, right: 0}` 을 가진다.
<br/>scrollIndicatorInsets props의 right 속성이 0일 경우, 아무런 값도 먹히지 않기 때문에 scrollIndicatorInsets props 속성을 0.1로 수정해 문제를 해결했다.

```tsx
<FlatList
  scrollIndicatorInsets={ㅤ{ right: 0.1 }}
  style={{ backgroundColor: theme.bg }}
  contentContainerStyle={{ paddingBottom: vw(100) }}
  ListEmptyComponent={
    <NullContentBox>
      <Photo
        w={277}
        h={160}
        source={require("../assets/myHome/empty-list.png")}
      />
      <BoldText size={40} lineHeight={54} mt={48}>
        상품이 없어요
      </BoldText>
    </NullContentBox>
  }
  data={productList}
  keyExtractor={(_, index) => index.toString()}
  renderItem={renderItem}
/>
```

## 정리

React Native로 개발하다보면 정말 다양한 오류들과 만날수 밖에 없다.
<br/>
내가 마주친 오류들은 이미 다른사람들도 겪어봤을 가능성이 높기 때문에, 오류가 발생했을 때 공식 문서나 이슈에 관련된 문서를 적극적으로 찾아 해당 내용을 정리하는 것이 중요한 것 같다.
<br/>실제로 오류가 발생할때마다 노션에 정리해두어 오류메시지가 어디서 본 느낌이 든다면 노션으로 검색해서 빠르게 오류를 해결한 경험이 있다.
<br/>React Native는 기본적으로 Javascript를 사용하여 Native Bridge로 Native 모듈을 연결하는 과정이 있기 때문에 다른 크로스 플랫폼 앱보다는 속도나 성능이 느릴 수 있다.<br/>그렇기 때문에 많이 사용하는 리스트나 이미지 같은 사소해 보일수 있는 부분이라도 성능을 개선하고 속도를 끌어올릴 수 있는지 고민해보는 시간이 필요하다.

{: .prompt-info }

> <span style="color:red">(2023.11 추가 완료)</span> [참고 코드 링크](https://github.com/Roen77/react-native-example)

{: .prompt-info }
