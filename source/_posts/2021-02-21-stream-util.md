---
layout: post
title:  "Stream Util"
date:   2021-02-21 00:0054 +0900
categories: java
---

### int[] -> List<Integer>
```java
Arrays.stream(new int[] {0, 1, 2, 3}).boxed().collect(Collectors.toList())
```

### List<Integer> -> int[]
```java
List.of(0, 1, 2, 3).stream().mapToInt(Integer::intValue).toArray()
```

### Swap을 활용한 Permutation
```java

public perm(int[] arr, int depth) {
    if(depth == arr.length) {
        print(arr, arr.length);
        return;
    }

    for(int i=depth; i<arr.length; i++) {
        swap(arr, i, depth);
        perm(arr, depth + 1);
    }
}

public void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}

public void print(int[] arr, int k) {
    for(int i=0; i<k; i++) {
        System.out.print(arr[i] + ", ");
    }
    System.out.println("");
}

```

### Prefix를 활용한 Permutation

```java

    public void perm(String pre, String src) {
        System.out.println(pre);

        for(int i=0; i<src.length(); i++) {
            perm(pre + src.charAt(i), src.substring(0, i) + src.substring(i+1));
        }
    }

```

### String -> char[]

```java
char[] chars = str.toCharArray();
```

### isPrime
```java

    public boolean isPrime(int num) {
        if(num == 1) return false;
        if(num == 2) return true;
        if(num % 2 == 0) return false;

        for(int i=3; i<=(int)Math.sqrt(num); i+=2){
            if(num%i==0) return false;
        }

        return true;
    }

```


### Math.max(triangle[i][j], triangle[i][j + 1]);


### 배열 자를때 문자열로 변환해서 하면 쉽다 배열 변환시 String.valueOf 사용 혹은 Arrays.toString


### int[] -> String[]

```java
Arrays.stream(numbers).boxed().map(String::valueOf).collect(toList())
```

### String[] -> String

```java
Collectors.joining()
```

### java 배열을 뒤집어주는 기능 

```java
        Collections.reverse(list);
```

### 테스트 셋의 종류

 null을 넣는다던지, 엄청 큰값을 넣는다던지..원소가 하나만 있다던지, 음수값, 오버플로우값, 비어있는 배열에 접근, 초기값이나 마지막 값이 이벤트인 경우


 ### Dynamic Programming - 카데인 알고리즘 (부분 배열의 최대 합 구하기)

 ### 유클리드 알고리즘

 ```java
 /* a가 더 큰값 */
private int gcd(int a, int b) {
    if(a % b == 0) {
        return b;
    }
        
    return gcd(b, a%b);
}

 ```