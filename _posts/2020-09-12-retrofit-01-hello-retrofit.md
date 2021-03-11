---
title: Retrofit 01 - Hello, Retrofit
author: Tink
layout: post
---

# Hello, Retrofit

一个简单的使用 Retrofit 访问 Restful API 例子

### 1. 给 project 添加 gradle 依赖

```kotlin
//
def retrofit_version = "2.9.0"
implementation "com.squareup.retrofit2:retrofit:$retrofit_version"
implementation "com.squareup.retrofit2:converter-gson:$retrofit_version"
```

Android 项目默认有两个 build.gradle 文件，一个是 Project 级别的，一个是 module 级别的，我们需要添加在 module 级别的 gradle

### 2. 给项目添加网络访问权限

```xml
// AndroidManifest.xml
<manifest...>
    <uses-permission android:name="android.permission.INTERNET"/>
</manifest>
```

### 3.了解我们的 API

我们使用一个简单的 Restful API，不需要任何的 key 就可以使用，一个货币汇率转换 API

[https://exchangeratesapi.io/](https://exchangeratesapi.io/)

一个简单的使用：获取当前基于美元的人民币汇率

[https://api.exchangeratesapi.io/latest?base=USD&symbols=CNY](https://api.exchangeratesapi.io/latest?base=USD&symbols=CNY)

其中https://api.exchangeratesapi.io 这部分是 Base URL

`latest`是 path

`?`后面是 Query

```java
{"rates":{"CNY":6.5113521695},"base":"USD","date":"2021-03-10"}
```

### 4.构建 Retrofit Service 和 Instance

```kotlin
// ConverterService.kt
interface ConverterService {
    @GET("/latest")
    suspend fun getRate(
            @Query("base") base: String,
            @Query("symbols") symbols: String)
    : ResponseBody
}
// 默认地，Retrofit会把HTTP body反序列化为OkHttp的ResponseBody

// RetrofitInstance.kt
object RetrofitInstance {
    val api: ConverterService by lazy {
        Retrofit.Builder()
                .baseUrl("https://api.exchangeratesapi.io")
                .build()
                .create(ConverterService::class.java)
    }
}
```

### 5. 在一个 IO 线程中调用

```kotlin
class ConverterViewModel: ViewModel() {
    var text = MutableLiveData<String>()

    init {
        text.value = "loading"
    }

    fun getRate(base: String, target: String) {
        viewModelScope.launch(Dispatchers.IO) {
            val response = RetrofitInstance.api.getRate(base, target)
            text.postValue(response.string())
        }
    }
}
```

[https://github.com/TinkZhang/RetrofitDemo/tree/feature/hello_retrofit_currency_converter](https://github.com/TinkZhang/RetrofitDemo/tree/feature/hello_retrofit_currency_converter)
