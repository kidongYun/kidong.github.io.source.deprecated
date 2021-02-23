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