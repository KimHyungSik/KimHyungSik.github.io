---
layout: post
title:  Android Navigation DeepLink 처리하기
categories: [Android]
tags: [Android, Navigation]
comments: true
---

참고 링크 : [링크 만들기](https://developer.android.com/guide/navigation/navigation-deep-link)

명시적 딥 링크 만들기 
==============
 PendingIntent를 사용하여 Navigation Deep Link를 생성할 수 있습니다. 명시적 딥 링크를 사용하면 네비게이션 스택을 앱에 들어간 이후 해당 화면 으로 이동까지의 탐색을 추가 합니다.
 기본적으로 PendingIntent를 생성 할 때는 NavDeepLinkBuilder 또는 findNavController().createDeepLink()를 사용하여 createPendingIntent()를 통해 명시적인 딥링크를 생성 할 수 있습니다.
 `ComponentName`이 있으면 `.setComponentName`을 사용하여 전달 할 수 있습니다.
 
```kt
val pendingIntent = NavDeepLinkBuilder(context)
    .setGraph(R.navigation.nav_main_graph)
    .setDestination(R.id.subFragment)
    .createPendingIntent()
    
// OR

val pendingIntent = findNavController()
              .createDeepLink()
              .setGraph(R.navigation.nav_main_graph)
              .setDestination(R.id.subFragment)
              .createPendingIntent()
```

암시적 딥 링크 만들기
===========
암시적 딥 링크의 경우 딥 링크가 호출되면 Android에서 앱을 열 수있습니다.
Navigation을 사용 한다면 navgigation graph에 deepLink 태크를 추가하여 간단하게 deepLink를 만들 수 있습니다.
```kt
<fragment
    android:id="@+id/subFragment"
    android:name="com.todotracks.savedstatehandle.ui.sub.SubFragment"
    android:label="SubFragment"
    tools:layout="@layout/sub_fragment"
    >
    <deepLink
        android:id="@+id/deepLink"
        app:action="android.intent.action.VIEW"
        app:uri="subfragment"/>
</fragment>
```
스키마를 생략하게 되면 http 또는 https로 가정하여 스키마를 생성합니다. {} 괄호를 사용하여 매개변수를 넘길 수 있습니다. `http://www.example.com/users/{id}?myarg={myarg}` 처럼 쿼리를 사용 할 수도 이씃빈다.
암시적 딥 링크를 설정 하려면 `manifest.xml`에도 설정을 추가 해야 합니다. navigation graph를 사용하는 요소를 activity에 `<nav-graph>`요소를 추가 해야 합니다.
```kt
<activity
    android:name=".MainActivity"
    android:exported="true">
    <nav-graph android:value="@navigation/nav_main_graph" />
</activity>
```

데이터 전달 및 수신
============
데이터 송신
----------
명시적 딥 링크를 사용 할 때 `setArguments`를 사용 하여 데이터를 넘길 수 있습니다.
```kt
val pendingIntent = findNavController()
              .createDeepLink()
              .setGraph(R.navigation.nav_main_graph)
              .setDestination(R.id.subFragment)
              .setComponentName(MainActivity::class.java)
              .setArguments(bundleOf("data" to "dataTEST"))
              .createPendingIntent()
```
혹은 암시적 딥 링크를 사용할 때는 경로 매개변수를 넘겨 데이터를 호출 할 수 있습니다.

데이터 수신
-------
데이터를 수신 할때는 arguments에서 같은 key값을 통해 데이터를 수신 할 수있습니다.
```kt
val data = arguments?.getString("data")
```
viewModel에서 `savedStateHandle`에서 key 값을 사용하여 데이터를 수신 할 수 있습니다.
```kt
val data = savedStateHandle.get<String>("data")
```

딥 링크 테스트
============
에뮬레이터를 이용하여 딥링크를 실행해 볼 수 있습니다. 
```
adb shell am start -W -a android.intent.action.VIEW -d "딥링크 주소" 패키지명

// example
adb shell am start -W -a android.intent.action.VIEW -d "http://subfragment/DeepLinkTest" com.todotracks.savedstatehandle/.MainActivity
```
