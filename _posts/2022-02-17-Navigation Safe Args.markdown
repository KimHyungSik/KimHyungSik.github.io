---
layout: post
title:  Navigation Safe Args를 이용하여 데이터 전달 하기
categories: [Android]
tags: [Android]
comments: true
---
Navigation으로 Fragment간에 데이터를 전달하는 방법에 대해 공부하던 도중 safe args를 알게 되어 정리 해 봅니다.

Safe Args를 사용하면 기존의 bundle를 이용하여 데이터를 전달 할떄 보다 안정적으로 데이터를 전달 할 수 있습니다.
먼저 Project단위에 classpath를 추가 합니다.
```
buildscript {
    repositories {
        google()
    }
    dependencies {
        def nav_version = "2.4.0"
        classpath "androidx.navigation:navigation-safe-args-gradle-plugin:$nav_version"
    }
}
```
앱 또는 모듈 단위의 build gradle에 plugins를 추가 합니다.
```
plugins {
  id 'androidx.navigation.safeargs'
  // OR
  id 'androidx.navigation.safeargs.kotlin'
}
```

Navigation Graph에서 Fragment에 전달 하고 싶으 Argumnet를 선언 하여 Argument를 전달 할 수 있도록합니다.
```
<fragment
    android:id="@+id/mainFragment"
    android:name="com.test.example.ui.main.MainFragment"
    android:label="main_fragment"
    tools:layout="@layout/main_fragment" >
    <action
        android:id="@+id/action_mainFragment_to_subFragment2"
        app:destination="@id/subFragment" />
</fragment>
<fragment
    android:id="@+id/subFragment"
    android:name="com.test.example.ui.sub.SubFragment"
    android:label="SubFragment"
    tools:layout="@layout/sub_fragment"
    >
    <argument
        android:name="name"
        app:argType="com.test.example.ui.sub.DataStr"
        />
</fragment>
```

DataStr은 단순 data class 입니다. Parcelable를 사용하여 데이터를 전달 할 수 있도록 정의 했습니다.
기본적을 제공되는 type은 [Android Developers](https://developer.android.com/guide/navigation/navigation-pass-data?hl=ko)에서 확인 할 수 있습니다.
```kt
data class DataStr(
    val str: String
) : Parcelable {
    constructor(parcel: Parcel) : this(parcel.readString().toString()) {
    }

    override fun describeContents(): Int {
        return 0
    }

    override fun writeToParcel(p0: Parcel?, p1: Int) {
        p0?.writeString(str)
    }

    companion object CREATOR : Parcelable.Creator<DataStr> {
        override fun createFromParcel(parcel: Parcel): DataStr {
            return DataStr(parcel)
        }

        override fun newArray(size: Int): Array<DataStr?> {
            return arrayOfNulls(size)
        }
    }
}
```

데이터를 전달 할 준비가 끝났습니다. 이제 Navigation을 이용 하기만 하면 됩니다.
데이터를 전달하는 과정은 심플합니다.
Navigation에서 생성된 함수인 ```MainFragmentDirections```를 사용하여 action를 불러오고 전달해야하는 데이터를 넣어 주면 끝입니다.
```
val directions = MainFragmentDirections.actionMainFragmentToSubFragment2(DataStr("new Data"))
findNavController().navigate(directions)
```

전달 받은 데이터를 받는 방법은 ```by navArgs```를 이용하여 데이터를 받을 수 있습니다.
```kt
private val arg : SubFragmentArgs by navArgs()
Log.d("로그", "onCreateView: ${arg.name.str}")
```

또는 전달 받은 데이터를 Hilt와 SavedStateHandle을 이용 하여 ViewModel에서도 받을 수 있습니다.
관련 해서는 [Passing Safe Args to your ViewModel with Hilt](https://mattrobertson333.medium.com/passing-safe-args-to-your-viewmodel-with-hilt-366762ff3f57)글을 읽어 보면 간단하게 구현 할 수 있습니다.
