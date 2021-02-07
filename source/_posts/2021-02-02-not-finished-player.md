---
layout: post
title:  "코딩테스트 - 완주하지 못한 선수"
date:   2021-02-02 00:0054 +0900
categories: algorithm
---

### 문제

수많은 마라톤 선수들이 마라톤에 참여하였습니다. 단 한 명의 선수를 제외하고는 모든 선수가 마라톤을 완주하였습니다.

마라톤에 참여한 선수들의 이름이 담긴 배열 participant와 완주한 선수들의 이름이 담긴 배열 completion이 주어질 때, 완주하지 못한 선수의 이름을 return 하도록 solution 함수를 작성해주세요.

제한사항
마라톤 경기에 참여한 선수의 수는 1명 이상 100,000명 이하입니다.
completion의 길이는 participant의 길이보다 1 작습니다.
참가자의 이름은 1개 이상 20개 이하의 알파벳 소문자로 이루어져 있습니다.
참가자 중에는 동명이인이 있을 수 있습니다.
입출력 예
participant	completion	return
[leo, kiki, eden]	[eden, kiki]	leo
[marina, josipa, nikola, vinko, filipa]	[josipa, filipa, marina, nikola]	vinko
[mislav, stanko, mislav, ana]	[stanko, ana, mislav]	mislav
입출력 예 설명
예제 #1
leo는 참여자 명단에는 있지만, 완주자 명단에는 없기 때문에 완주하지 못했습니다.

예제 #2
vinko는 참여자 명단에는 있지만, 완주자 명단에는 없기 때문에 완주하지 못했습니다.

예제 #3
mislav는 참여자 명단에는 두 명이 있지만, 완주자 명단에는 한 명밖에 없기 때문에 한명은 완주하지 못했습니다.

### 솔루션

```java

import java.util.HashMap;
import java.util.Map;

class Solution {
    public String solution(String[] participant, String[] completion) {
        Map<String, Integer> map = new HashMap<>();

        for(int i=0; i<participant.length; i++) {
            map.put(participant[i], map.getOrDefault(participant[i], 0) + 1);
            if(i != participant.length - 1) {
                map.put(completion[i], map.getOrDefault(completion[i], 0) - 1);   
            }
        }

        String result = null;
        for(String key : map.keySet()) {
            if(map.get(key) != 0) {
                result = key;
            }
        }

        return result;
    }
}

```

### 포인트

이 문제를 추상화 시키면 두 배열을 비교하는데 이 중에 서로 맞지 않는 값을 찾아내는 문제다.
추가적으로 중복이 가능하다는 조건이 있다.

이를 해결하기 위한 포인트는 그 값들을 세서 해쉬에 저장하는 방법. 이였다.