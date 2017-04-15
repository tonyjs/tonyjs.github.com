# Single Activity Architecture ?

2012년부터 본격적으로 안드로이드 개발을 시작하고 가장 많이 공부한 것이 `Fragment` 에 대한 것이었습니다. 처음 시작하는 안드로이드 개발자라면 그 누구라도
`안드로이드 4대 컴퍼넌트`에 대해 열심히 공부하겠지만,
제가 처음으로 만든 앱이 폰버전, 태블릿버전 따로 나뉜 프로덕트였던 지라 너무나도 자연스럽고 당연하게 Fragment에 대한 공부에 가장 많은 시간을 할애했죠.

특히나 태블릿에서는 하나의 `Activity`에 여러 Fragment를 다루는 것을 자주 했었습니다. 큰 틀에서 보면 다양한 디바이스를 지원하는 것이 개발자나 디자이너의
실력 향상에도 매우 좋고 사용자 측면에서도 이만큼 좋은게 없어서 아름다운 일이라고 생각하지만, 이는 많은 지식과 넓은 생각 그리고 많은 시간을 요구로 하는데
우리에게 그런 것들이 있을 수 없기 때문에(특히나 시간은...) 시간이 흐르고 여러가지 프로젝트를 하면서 태블릿에 대한 지원은 점점 더 멀어져 가게 됐습니다.
(새 프로젝트 만들면 `values-w820dp` 이런 큰 기기용 폴더부터 지우게 됨..) 그래서 자연스럽게 하나의 Activity에 두 개 이상의 Fragment를
다루는(`ViewPager` 빼고)일이 드물어 졌습니다.

점점 Activity는 껍데기 용으로 쓰이는 걸 느끼게 됩니다. 사용자의 요구, 기획적인 요구사항에 의해서 같은 화면을 다른 상황에서 다르게 보여줘야 될 일이 너무나
많기 때문에 Fragment를 안 쓸 수는 없고... 결국 다음과 같은 코드도 조금씩 생기기 시작합니다.
```
public class UserActivity ... {

    public void onCeate() {
        Intent intent = getIntent();
        id = intent.getExtra("id");
        name = intent.getExtra("name");
        profile = intent.getExtra("profile");
        sattus = intent.getExtra("status");

        addFragment(
            UserFragment.newInstance(id, name, profile, status);
            // or
            UserFragment.newInstance(intent.getExtras);
    }
}

public class UserFragment {

    public void onViewCreated() {
        Bundle args = getArgument()
        args.getLong("id");
        args.getLong("name");
        args.getLong("profile");
        args.getLong("status");
    }
}
```
같은 일을 두번하거나 `Bundle`객체가 두번에 거쳐 전달된다던가 도 문제라고 생각이 들었지만 하나의 도메인에 대한 화면을 구성하기 위해서 위대한 안드로이드 1대 컴퍼넌트가
큰 일안하고 껍데기로서 낭비되는 것이 안쓰러웠습니다. 이외에도 푸쉬나 스키마를 통해서 다수의 Activity를 띄워야 하는 상황, 다양한 화면 `Transition`에 대한 요구들
등 때문에

화면을 전환을 담당하는 하나의 Activity 와 여러가지 `View`의 조합으로 어플리케이션을 구성하는 `Single Activity Architecture` 에 관심을 가지게 됩니다.

### Conductor ?

`View`의 역할에 Fragment를 대입하는 방식으로 작업을 하려다 `Single Activity Architecture` 를 위한 여러가지 라이브러리 들이 있다는 것을 알게 됐습니다.
그중의 하나가 지금 다룰 [Conductor](https://github.com/bluelinelabs/Conductor)란 것인데요. [GoogleSample](https://github.com/grepx/android-architecture) 에서
이것의 [존재](https://github.com/grepx/android-architecture/tree/todo-mvp-conductor)를 알게 됩니다. 3자가 작성하긴 했지만 구글에서 제공해주는 샘플도 있겠다
점점 더 흥미로워지고 본격적으로 사용을 해보기 시작합니다.

(! 예제는 대세 `Kotlin`으로 쓰여집니다.)
```
class RouterActivity : AppCompatActivity() {

    lateinit var router : Router

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_router)

        router = Conductor.attachRouter(this, container, savedInstanceState)
        if(!(router.hasRootController())) {
            router.setRoot(RouterTransaction.with(MainController()))
        }
    }

    override fun onBackPressed() {
        if (!router.handleBack()) {
            super.onBackPressed()
        }
    }
}
```
여러가지 화면을 다루는 `Router` 용 Activity 구성이 끝났습니다. 쉽게 예를들면 Fragment나 Activity의 대체제로 `Controller`라는 녀석이 쓰이게
되는데요. 위 코드를 보통 방식으로 치환하면 `SplashActivity(LauncherActivity)` 에서 `MainActivity` 를 호출하는 것이 되겠습니다.
```
class MainController : Controller() {

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup): View {
        val view = inflater.inflate(R.layout.controller_main, conatiner, false)
        view.aButton.setOnClickListener {
            router.pushContrller(RouterTransaction.with(AController())
        }
        view.bButton.setOnClickListener {
            router.popContrller(RouterTransaction.with(AController())
        }
        view.cButton.setOnClickListener {
            getChildRouter(view.conatiner).pushContrller(RouterTransaction.with(CController())
        }
        return view
    }

}
```
`Controller`는 `Router`에 접근해서 다른 Controller를 보여줄 수도 있고(Activity가 다른 Activity를 호출하듯), `ViewGroup`을 통해 새로운
Router를 만들어서 자식 Controller를 보여줄 수도 있습니다.(Activity에서 Fragment를 보여주듯) 전반적인 메소드 네이밍을 보면 `FragmentTransaction`
에서 제공하는 `add`, `remove`, `show` 등등등 과 유사한 기능들을 제공하고 있습니다. 결론적으로는 Activity와 Fragment의 특성을 전부 가지고 있는 것이
Controller라고 할 수 있겠습니다.

Activity와 Fragment의 경계를 허물고 화면에 대한 디자인이 나오고 개발을 시작할 때 Activity로 할지 Fragment로 할지 고민하지 않아도 되어서 이것으로
Single Activity Architecture를 구현하면 되겠구나 했습니다만, 제대로 생산성있게 사용할라면 몇가지 공통 메소드 들을 추상화 시키고 써야 할텐데 이렇게
되면 대체 Fragment와 다른 점이 뭔가 싶어집니다. 그동안 개발했던 많은 프로덕트들 거의 전부 `BaseFragment`, `BaseActivity`들 만들어서 백버튼에 대한 제어,
`FragmentTransaction.add, remove, replace, show, hide` 등을 추상화해서 사용했는데 굳이 Conductor 내의 라이프사이클, 메소드 등을 공부할 필요가
있는가?

일단 두 가지를 `Conductor`를 쓰기 위한 합리화로 뽑았는데요. 첫번째는 자유로운 화면 `Transition` 구현에 대해서 Conductor가 훨씬 쉽다는 점.
Conductor에서는 화면 전환에 필요한 클래스를 추상 클래스 `ControllerChangeHandler`라고 정의하고 있고, 기본적으로 제공되는 `ChangeHandler`도 엄청
많이 있습니다. [기본적인것 외에 활용가능한 ChanglHandler들](https://github.com/bluelinelabs/Conductor/tree/develop/demo/src/main/java/com/bluelinelabs/conductor/demo/changehandler)
```
새로운 화면 진입시 아래서 위로, 이전 화면으로 돌아갈시 위에서 아래로 애니메이션되는 Transition 설정
router.pushController(RouterTransaction.with(UserController())
	.pushChangeHandler(VerticalChangeHandler())
	.popChangeHandler(VerticalChangeHandler()))
```
두번째는 기존 추상화된 클래스들을 더 깊이있게(상태저장 등) 사용하기 위해 수정해야 하는데 , Conductor에는 이미 구현되어 있는 것들이 많다.

이 두 가지 이유를 들어 새로운 프로젝트에 적용시키고 있는데요. 조금 부정적이지만 아래와 같은 결론도 도출 할 수 있었습니다.

*Activity, Fragment 이것들과 관련한 웬만한 이슈들의 해결 방법들은 금방 찾을 수 있고, 어느정도 노하우가 생겨서 `안드로이드 4대 컴퍼넌트` 안에 속한 이 둘을
사용하는 것에 의구심이 없다면 굳이 Conductor를 사용하지 않아도 된다.*

입니다. Conductor 라이브러리 걷고 다시 돌아가는데 많은 시간이 걸리진 않아서 "내가 지금 왜 사서 고생을 하고 있는가, 빨리 걷자" 라고 생각하면서도
이상하게 계속 쓰게 되는데요. 돌이켜보면 제가 가진 부정적인 포인트들이 Conductor 에 대한 것이 아니라 Single Activity Architecture를 향하고 있기
때문인 것 같습니다. 사서 고생이거나 고민중인 포인트 들은 다음과 같습니다.

1. Theme 지정 - 단일 Activity가 아니라면 `styles.xml`에 각 Activity마다 필요로 하는 색상값과 같은 화면성격을 기술해 놓으면 쉽게 적용 가능한데,
Single Activity Architecture를 적용시키면 하나 하나의 `View`마다 일일히 theme 설정이 필요함.

2. StatusBar - 1번과 마찬가지로 각 화면마다 필요한 상태바 관련 설정들을 쉽게 하지 못한다. 색상값은 해결이 가능하다 하더라도 어떤 화면에서는 완전 오버레이된
상태바가 필요하고 어떤 화면은 불투명으로 오버레이된 상태바가 필요할 때, 유연하게 적용시키기 어렵다. (`RouterActivity`를 완전 오버레이 시켜놓고 각 `Controller`
에서 대응하는 방법으로 회피 중)

3. SoftInput - 키보드가 올라옴에 따라 어떤 화면에선 resize 가 요구되고 어떤 화면에선 현재 상태를 유지하는 것이 요구되는 경우도 생길텐데, 이것도 유연하게 대응하기
힘들 듯(대부분의 경우 resize 가 요구되기 때문에 어...음...)

4. Kotlin - `Kotlin`을 메인 언어로 사용중인데 `android-extensions`나 `anko` 등이 Activity,Fragment에서 편리한 메소드들을 제공해주고 있는데,
Conductor를 사용하게 되면 그 좋은 메소드들을 사용하는데 제약사항이 생길 수 밖에 없음.

깍두기. Learning curve - 그렇게까지 높다고 생각되진 않지만, 안드로이드 앱 개발에서 가장 중요한 화면을 제어하는 라이브러리 인 만큼 Conductor의 라이프사이클
이나 메소드들을 공부할 필요가 있음

이 정도를 단점으로 꼽고 있는데요. 이 단점들이 Activity의 Task 와 Stack 에 대한 고민을 해소해주고, 화면에 대한 구성을 하나의 개념으로 통일화해서 개발하고, 
무분별한 Activity 사용으로 인한 복잡도를 줄여주는 등의 장점들을 상쇄할만큼으로 보이지만, 개인적으로는 충분히 시도해볼만한 가치는 있다라고 판단하고 있습니다.

중요한 것은 개발과 관련된 어떤 언어, 어떤 개념, 어떤 라이브러리 든 장단은 있으니, 맹목적이지 않고 유연한 사고를 하는 것임을 인지하고, 납득가능한 방법이 아닌 
회피로직을 만들 때는 "단일" 이란 개념은 생각하지 않도록 해야겠죠.

Conductor와 함께하는 Single Activity Architecture 어플리케이션 개발은 계속 될 것이고, 결과물이 나온 후 회고에 대한 블로그를 남기도록 하겠습니다.
감사합니다.

ps. 요즘 사용중인 [`BasePage`](https://gist.github.com/tonyjs/5bc7aa7914daeb0ab164c8d6d3d3c11f), `Controller`라는 이름이 낯설고, MVP 패턴 이용중에 네이밍이 좀 겹쳐서 `Page`로 화면단위 네이밍 중입니다.
