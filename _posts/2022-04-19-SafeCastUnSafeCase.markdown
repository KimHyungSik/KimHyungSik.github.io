---
layout: post
title:  UnSafe Cast와  Safe Cast
categories: [ Kotlin ] 
tags: [ Kotlin ]
comments: true 
---
Type Casting
===
type cast를 잘못 활용하여 문제가 되는 코드를 많이 접하면서 Type casting에대 다시 한번 공부해보았다. Type Cast를 잘 활용하여 짧은 코드로 가독성있는 코드를 작성해보자.

- as
- is
  
is, !is
---
Kotlin에서 is를 이용하면 명시적인 형변환 없이 smart cast를 이용하여 Type Cast를 할 수 있습니다. is또는 !is를 사용하면 련타임에 지정한 유형으로 변환이 가능한지 확인 하여 Type Cast를 진행 할 수 있습니다. !is는 Type Cast가 불가능 한경우 사용할 수 있습니다.
```kt
if (obj is String) {
    print(obj.length)
}

if (obj !is String) {
    print("Not a String")
} else {
    print(obj.length)
}
```
is는 when 또는 while과도 함께 사용 가능 합니다.
```kt
when (obj) { 
    is Int -> print(obj + 1) 
    is String -> print(obj.length + 1) 
    is IntArray -> print(obj.sum()) 
}
```

as, as?
----
as 는 Unsafe Type Cast로 변환이 불가능한 경우 cast불가 에러를 발생 시킵니다.
```kt
val x: String = y as String
```
만약 위의 코드에서 y가 null이라면 NPE를 발생 시킬 수 있습니다.
```kt
val x: String? = y as String?
```
nullable형식으로 수정이 필요합니다.

as? 는 Safe(Nullable) Type Cast입니다. 변환이 불가능하거나 null인 경우 에러가 아닌 null을 반환 합니다.
```kt
val x: String? = y as? String
```
