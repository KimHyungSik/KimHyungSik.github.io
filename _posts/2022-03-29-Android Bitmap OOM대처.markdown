---
layout: post
title:  Android Bitmap OOM대처
categories: [Android]
tags: [Android, Bitmap]
comments: true 
---

안드로이드 기본 지원 이미지 포맷 BMP, GIF, JPG, PNG, WebP, HEIF(8.0이후 지원)

메모리관리(Overview of memory management  |  Android Developers )

안드로이드의 앱에는 사용가능한 메모리가 할당되고 할당된 메모리 크기를 엄격히 제한 한다.
허용된 메모리를 넘기게 되면 OutOfMemoryError를 발생하며 앱이 종료될 수 있습니다.

6960 * 4640의 상당한 크기의 이미지로 인해 OOM이 발생하는 문제가 있었다. 6960 * 4640사이즈의 이미지는 bitmap의 기본 설정인 ARGB_8888로 셋팅하여 로드하게되면 129MB로 상당히 많은 메모리를 차지하게 된다.


이문제에 대해 Android Developer에서는 몇가지 해결책을 제시해 주고 있습니다.(Handling bitmaps  |  Android Developers )

가장 편한 방법으로 Glide를 이용하면 간단하게 비트맵을 가져와 디코딩하고 캐싱하는 작업을 진행할 수 있으며 Android Developer 에서 추천하는 방법 입니다.

다른 방법으로는 Bitmap관련 API를 이용하여 직접 작업하는 방법에대해서도 가이드 하고있습니다.
큰 비트맵을 효율적으로 로드 이 가이드를 통해 이미지 크기를 샘플링하여 화면에 보여주는 방식을 구현 해보았습니다.

```kt
val img = BitmapFactory.Options().run {    
    bufferedInputStream.mark(bufferedInputStream.available())
    inJustDecodeBounds = true    
    BitmapFactory.decodeStream(bufferedInputStream, null, this)
    
    inSampleSize = calculateInSampleSize(this, view.width, view.height)

    bufferedInputStream.reset()
    inJustDecodeBounds = false    
    BitmapFactory.decodeStream(bufferedInputStream, null, this)
}
```
view.setImageBitmap(img)
inJustDecodeBounds 옵션을 true로 주게 되면 bitmap이 메모리에 등록되지는 않지만 이미지의 크기, 타입등을 가져올 수있습니다.  크기를 확인하여 샘플링 할지 하지않을지 등을 선택할 수 있습니다.(이번 작업에서는 구현하지 않았습니다)

 
```
inJustDecodeBounds = true
```
설정으로 디코딩한 이후 
```
inSampleSize = calculateInSampleSize(this, view.width, view.height)
```
현제 뷰 크기에 맞게 샘플링할 사이즈를 계산해 줍니다.
```
inJustDecodeBounds = false
```
다시 설정을 변경하여 이미지를 샘플링 크기에 맞게 다시 디코딩 해줍니다.

calculateInSampleSize의 구현을 다음과 같이 되어있습니다.

```kt
fun calculateInSampleSize(options: BitmapFactory.Options, reqWidth: Int, reqHeight: Int): Int {
    val (height: Int, width: Int) = options.run { outHeight to outWidth }    
    var inSampleSize = 1    
    if (height > reqHeight || width > reqWidth) {
        val halfHeight: Int = height / 2        
        val halfWidth: Int = width / 2        
        while (halfHeight / inSampleSize >= reqHeight && halfWidth / inSampleSize >= reqWidth) {
            inSampleSize *= 2        
        }
    }
    return inSampleSize
}
```
이미지의 크기가 지정한 이미지 사이즈 보다 큰경우 2의 거듭제곱으로 샘플 크기를 계산합니다.
```
Glide.with(view).load(url).into(view)
```
간단하게 Glide로도 이미지 로드에는 문제가 없습니다.
