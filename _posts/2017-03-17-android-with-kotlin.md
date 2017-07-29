---
layout: post
title: Android with Kotlin
date: 2017-03-19
---

# Kotlin

### ***Kotlin is a statically typed language that targets the JVM and JavaScript. It is a general-purpose language intended for industry use.***

&nbsp;`JVM`, `Javascript` 를 대상으로 하는 언어라고 합니다.([기타 Kotlin 에 대한 설명들](https://kotlinlang.org/docs/reference/faq.html))

&nbsp;`Java` 로 안드로이드 개발을 하고 있는 저에게는, Java 가 아닌 다른 언어로 안드로이드 개발을 하는 것이 신기했고 또, 사용하면 할수록 묘한 매력이 있어서 2017년 부터는 Kotlin 위주로 코딩을 하고 있습니다.<br/>&nbsp;물론 저는 Java 로 된 안드로이드 프레임워크, 오픈소스 라이브러리 들을 "응용"하는 수준의 개발자이기 때문에, 아직까지는 자바의 어떤 부분이 안 좋고, 코틀린을 통해서는 어떻게 이런 것들이 해결될 수 있는지 깊게 알지 못합니다.<br/>&nbsp;그래서 `사용성`, `생산성` 위주로 Kotlin 으로 안드로이드 어플리케이션을 개발 했을 때 느꼈던 점들을 공유할까 합니다.

문법적인 부분에서 편해진 부분들이 많이 있는데 이런 부분들을 간단히 살펴보도록 하겠습니다. <br/>([Kunny 님 블로그](http://kunny.github.io/), [권태환 님 블로그](http://thdev.tech/) 등에 좀 더 자세히 소개 되어 있습니다.)

새로운 액티비티 시작 할 때를 예로들면,
<br/>`Java`
```
Intent intent = new Intent();
intent.putExtra(...);
intent.putExtra(...);
intent.addFlags(...);
startActivity(intent);
```
`Kotlin`
```
startActivity(Intent().apply {
    putExtra(...)
    putExtra(...)
    putExtra(...)
    addFlags(...)
})
```
보시다 싶이 `Kotlin` 이 더 간결한 형태를 보입니다. `Java` 에서도 빌더패턴을 만들어준다면 비슷하게 구현 할 수 있지만, Kotlin 에서는 `apply` 구문 하나로 어떤 객체라도 내부에 필요한 데이터들을 담을 수 있습니다.

`Fragment` 에서 `Activity` 접근에 대한 방어 코드로 `Null Check` 를 자주 하는 편인데,
Java 에서는 이런 부분마다 `if` 문으로 걸러줘야 했습니다.
```
if(getActivity() == null) {
    return;
}
getActivity().finish();
// or
if(getActivity() != null) {
    getActivity().finish();
}
```

`Kotlin` 에서는 금방 끝납니다.
```
activity?.finish() // ? 가 null check 를 대신해줌

* getActivity() 를 activity 로 표현 가능
```

`let` 구문을 사용하면 다음과 같은 형태로도 사용가능합니다.
```
activity?.let {
    it.setSupportActionBar(...)
    it.registerReceiver(...)
    it.getSupportFragmentManager().beginTransaction(...)
```

자동 형변환 기능도 맘에 듭니다.
<br/>`Java`
```
Activity activity = getActivity();
if(activity instanceOf StatusBarManager) {
    ((StatusBarManaget)activity).setStatusBarColor(...);
```
`Kotlin`
```
val activity = activity
if(activity is StatusBarManager) activity.setStatusBarColor(...);
```

Kotlin 문법들을 좀 더 많이 활용해보고 좋은 것들을 뽑아서 최대한 "Java 스럽지 않은" 스타일로 개발 해보려 하고 있습니다.

#### 어플리케이션 개발을 `Kotlin`으로...

Kotlin 을 사용하여 비교적 작은 덩치의 앱을 만들었는데요. 이 과정에서 느꼈던 부분들을 공유합니다.(공개용 repository 는 준비중에...)

먼저 기존과는 다른 언어 Kotlin 을 사용함으로서 생긴 문제는 거의 없었습니다. 언어적인 트러블이 거의 없었다는 걸로 보아 러닝커브가 그리 크지 않은 듯 합니다. 만약 러닝커브가 제 생각보다 큰게 일반적인 개발자들의 반응이라면, 언어적인 트러블을 겪기엔 난이도가 그리 높지 않은 앱을 만들었던 듯합니다.

설계패턴적인 부분에선, `MVP` 패턴을 그동안 계속해서 사용해왔었는데, 덕분에 문법적인 부분을 습득하기 좋았습니다. Kotlin 으로 구현해서 얻는 이득도 많았구요. 툭성에 떄라 구현하는 부분이 나뉘어 있으니 기존에 Java 로 구현된 코드와도 위화감 없이 비교할 수 있었고, 조금 더 구체적으로<br/><br/>&nbsp;
`View` 를 구현하는 `Activity/Fragment` 에서는 `let`, `apply`, `null-safe` 등의 사용으로 깔끔해졌고. <br/><br/>&nbsp;`Model` 에서는 메소드를 더 간결하게 만들고 콜렉션등을 다룰 때 좋았습니다.(`stream api` 이용한 Iteration 등), <br/><br/>&nbsp;`Presenter` 에서는 케이스 단위 작업까지 더해서 전체적인 언어적 이점을 활용하기 좋았습니다.

&nbsp;또 유명한 오픈소스 라이브러리들을 아무런 문제없이 이용할 수 있엇습니다. Rx 와 관련해서는 앞으로 어떻게 할까 고민이 생겼는데요. Kotlin 언어에서 자체적으로 지원해주는 것들과 [coroutines](https://github.com/Kotlin/kotlin-coroutines/blob/master/kotlin-coroutines-informal.md#use-cases) 를 같이 활용하는것이 Rx를 이용하는 것보다 더 깔끔해질 수 있다면 앞으로는 라이브러리를 사용안해도 될런지 싶습니다.

#### 음... 코드를 보고 싶은데...?

백마디 말보다 소스코드 한 줄이 더 이해가 빠르다고, 소스코드 공유를 통해 좀 더 많은 부분을 공유하고 피드백도 받고 싶습니다.
공개용 Repository 를 조만간 완성해서 Kotlin 과 함께하는 여정을 계속 공유 할 예정입니다.

#### 앞으로도 `Kotlin` ?
큰 프로젝트에 앞서 자그마한 프로젝트로 Kotlin 을 다뤄 봤는데요. 사실 "`Java` 도 통달하지 못한 상태" 에서 다른 언어를 사용하는게 맞는건지에 대한 망설임 과 레퍼런스가 적다는 두려움 이 있지만, 칼을 한 번 뽑았으니 무라도 썰어보자는 마음입니다. 지속적인 스터디로 `Kotlin` 과 함께하려고 합니다 :)
