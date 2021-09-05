---
title: "기수법, int ↔ String 변환"
---

# 
## 요약
- **int →String 변환하면서 진법 변환**
	- Integer.toString(int i, int radix): i에 변환하고 싶은 값, radix에 기수(몇 진수인지) 입력
- **String → int 변환하면서 진법 변환**
	- Integer.parseInt(String s, int radix): s에 변환하고 싶은 문자열, radix에 기수 입력
	- Integer.valueOf(String s, int radix) 도 가능
<br>
## 기수법 변환
### 
대상 수/기수 = 0이 될 때까지 계속 나누고, 나머지를 적어준다. 나중에 나온 나머지부터 차례대로 연결하면 해당 기수로 변환된 값을 얻을 수 있다.

예를 들어, 13~10~을 2진수로 변환한다면 아래와 같은 과정을 거치면 된다.
<table border = "2">
<tr>
<td>계산</td><td>결과</td><td>나머지</td>
</tr>
<tr>
<td>13/2</td><td>6</td><td>1</td>
</tr>
<tr>
<td>6/2</td><td>3</td><td>0</td>
</tr>
<tr>
<td>3/2</td><td>1</td><td>1</td>
</tr>
<tr>
<td>1/2</td><td>0</td><td>1</td>
</tr>
</table>

따라서 13~10~ = 1101~2~임을 알 수 있다.

### 구현: Java
```java
public static String convertIntToStringV1(int target, int radix) {  
    String result = "";  
  
    while (target>0) {  
        result += target % radix;  
        target /= radix;  
    }  
  
    StringBuilder sb =new StringBuilder(result);  
    return sb.reverse().toString();  
}
```
위 방식으로 구현한 자바 코드다.
먼저 나눴을 때 남은 나머지부터 계산하기에 (위의 13~10~을 2진수로 변환하는 경우, 1101~2~가 아니라 1011~2~로 변환되기에) StringBulider를 이용하여 문자 순서를 한 번 뒤집어주었다.

## n진수를 10진수로 다시 변환하는 방법

1101~2~ = 1 * 2^0^ + 0 * 2^1^ + 1 * 2^2^ + 1 * 2^3^ = 13~10~
이므로,

n진법으로 표현된 수
abcde...j~n~ = j * n^0^ + i * n^1^ + ... + a* n^9^ 형태가 된다.

### 구현: Java
```java
public static int convertStringToIntV1(String target, int radix) {  
    int result = 0;  
  
    for (int i = 0; i < target.length(); i++) {  
        result += (target.charAt(i)-48) * pow(radix, i);  
    }  
  
    return result;  
}
```
Math.pow를 이용하여 radix^i^를 구하고, 각 자리수에 곱해서 합했다.
자리수별 문자를 charAt(i)을 구한 다음에 -48을 한 이유는
ASCII코드 상 0이 48, 1이 49, ...이기 때문에 charAt(i)로 구한 숫자 문자 값을 숫자 값으로 바꿔주기 위함이다.

## Java가 제공하는 메서드를 이용하는 방법
### Integer.toString(int i, int radix)
`Integer.toString(int i, int radix)` 메서드는 int i를 radix 진법으로 변환하여 String 형태로 반환한다.
`Integer.toString(13, 2)`를 출력해 보면 String "1101"을 반환하는 것을 볼 수 있다.

### Integer.parseInt(String s, int radix) 혹은 Integer.valueOf(String s, int radix)
위 두 메서드는 반대로 radix 진법으로 표현된 String s를 int 값으로 변환시킨다.
`Integer.parseInt("1101", 2)`를 출력해보면 13을 반환하는 것을 볼 수 있다.

## NumberFormatException
그런데, `Integer.toString`으로 변환한 String을 `Integer.parseInt`를 이용하여 위의 변화했던 radix보다 낮은 radix로 변환할 경우 `NumberFormatException`이 발생한다. (`Integer.parseInt(Integer.toString(15,16), 14)` 와 같은 경우)` 물론 실제로 이런 식으로 변환할 일은 사실상 없긴 하다.

**1.  radix 값이 String이 표현하는 수를 포함하지 못할 때**

이는 더 낮은 radix의 경우 숫자로 인식할 수 없는 것이 존재하기 때문이다. 가령 0~7을 사용하는 8진법에서 숫자 8이나 9를 인식할 수는 없다.

**2. String s = null 일 때**
또한 변환하고자 하는 String s가 null 일때도 당연히 Exception을 던진다.

**3. radix 값이 `Character.MIN_RADIX`보다 작거나 `Character.MAX_RADIX`보다 클 때**
다룰 수 있는 최소 radix값(`MIN_RADIX`, 2)보다 작거나 최대 radix값(`MAX_RADIX,` 36)보다 클 경우에도 Exception을 던진다.

`MIN_RADIX` 값이 2인 것은 이해할 만 하지만 `MAX_RADIX` 값이 36인 것은 설명이 더 필요하다.
이는 10이상의 수를 표현할 때 a, b, c, d, e, ... , z로 표현하기 때문인데, z로 표현할 수 있는 수가 36이기 때문에 그 이상은 다루지 않는다.

### `Integer.ToString(int i, int radix)`의 radix 처리
반면 `Integer.toString()`은 `MIN_RADIX`값보다 작거나 `MAX_RADIX` 값보다 큰 radix 값을 받아도 `NumberFormatException` 발생시키지 않고 변환하는데, `java.lang.Integer.java`를 살펴보면 다음과 같은 코드를 볼 수 있다:
```java
public static String toString(int i, int radix) {  
    if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)  
        radix = 10;  
  
  /* Use the faster version */  
  if (radix == 10) {  
        return toString(i);  
  }
...
```
즉 radix가 범위를 초과할 경우 기수를 10으로 대입해 버리며, 기수가 10일 경우 그냥 진법 변환 계산이 필요 없는 `toString(i)`를 다시 호출하여 속도 향상을 꾀한다는 것을 알 수 있다.

`toString(int i, int radix)`는 진법 변환 시에 `digits[]`를 참조하는데 이는 다음과 같이 정의되어 있다:
```java
static final char[] digits = {  
    '0' , '1' , '2' , '3' , '4' , '5' ,  
  '6' , '7' , '8' , '9' , 'a' , 'b' ,  
  'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,  
  'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,  
  'o' , 'p' , 'q' , 'r' , 's' , 't' ,  
  'u' , 'v' , 'w' , 'x' , 'y' , 'z'  
};
```
이 36개의 `char`로 수를 표현하기에 `MAX_RADIX`값이 36인 것이다.
