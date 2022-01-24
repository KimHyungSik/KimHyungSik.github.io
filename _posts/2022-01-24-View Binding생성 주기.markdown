---
layout: default
title: View Binding by lazy 주의 하기
subtitle: View Binding by lazy 주의 하기
categories: Android
tags: [android, view binding]
comments: true 
---

view binding으로 작업을 하던 도중 무심코 이런 코드를 작업한적이 있다. viewHelper에 viewBinding을 넘겨 다른 작업을 class로 분리할려고 생각했었다.
```kt
private val viewHelper : ViewHelper by lazy {
  ViewHelper(binding) // binding은 viewbinding을 가져온 것 입니다.
}
``` 
한마디로 완전 잘 못된 코드이다.

fragment로 작업하여 다른 화면으로 갔다가 돌아오게 된다면 fragment의 lifecycle에 따라 onCreateView 부터 다시 실행 되게 되는데
```kt
override fun onCreateView(
    inflater: LayoutInflater, container: ViewGroup?,
    savedInstanceState: Bundle?
): View {
    _binding = FragmentWewootDealsRequestListBinding.inflate(inflater, container, false)
    return binding.root
}
``` 
이과정에서 생성되는 binding의 주소가 달라 지게됩니다. 생성된 binding의 hashcode를 확인했더니 완전 다른 주소로 이전에 생성된 binding과는 다른 객체를 가지고 있습니다.
``` 
로그: onCreateView: 242219912
로그: onCreateView: 106868090
``` 
하지만 by lazy로 생성된 viewHelper함수에는 이전에 생성된 binding값이 그대로 들어있게 되어 첫 fragment의 viewHelper는 정상적으로 작동하지만 이 후 한번 화면을 이탈하지만 fragment가 완전히 사라지기전에 fragment로 돌아 오게 되면 
이미 생성된 viewHelper는 다른 binding의 주소를 가지고 있게되어 GC가 처리하지 못하게 되며, 제대로 화면 또한 조작 할 수 없게 됩니다.
