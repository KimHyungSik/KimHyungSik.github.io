---
layout: post
title:  Android LiveData PostValue와 SetValue
categories: [ AAC, Android ] 
tags: [ AAC, Android, LiveDatat ]
comments: true 
---

Android에서 ViewModel 사용하면 LiveData를 사용한 Observer 패턴을 많이사용한다. LiveData는 `setValue()`, `postValue()` 두 메서드를 제공하며 두 메소드 모드 LiveData의 값을 변경 시키는 동작을 수행합니다. 그렇다면 두 메서드는 무슨 차이가 있을까요

setValue()
-----
Value를 Set하는 함수로만 생각하면 됩니다. 메인 쓰레드에서 값을 변경시켜 옵저빙 하고있는 객체에 변경사항을 반영시킨다.

postValue()
----
메인 쓰레드가 아닌 다른 쓰레드에서 LiveData의 값을 변경시키고 싶은 경우에 사용 되며 내부구현을 확인하면 post를 MainThread로 보내서 동작하므로 setValue와 동작 자체는 비슷하다고 생각하면 된다. 하지만 비동기 적으로 동작하므로 즉시 반영 되지않는 점을 유의하여야 한다.
