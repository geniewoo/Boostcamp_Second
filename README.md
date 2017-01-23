# Lesson2
### EmptyDatabase
listView의 setEmptyView로 만약 listView가 비어있을 때 보여줄 view를 정한다.
### Are We Offline
listview에서 emptyView를 보여 줄 때 왜 보여주는지는 말해주지 않는다. 그래서 왜 보여주는지까지 알려주어야한다.
    - user가 잘못된 인풋을 넣엇을 때, empty Response, non-JSON response, internet 이 안될때
    - android.permission.ACCESS_NETWORK_STATE 추가.
    - 인터넷 연결중인지 확인하는 코드, permission필요하다.
    ~~~
        ConnectivityManager cm =
                (ConnectivityManager)c.getSystemService(Context.CONNECTIVITY_SERVICE);

        NetworkInfo activeNetwork = cm.getActiveNetworkInfo();
        return activeNetwork != null &&
                activeNetwork.isConnectedOrConnecting();    
    ~~~
### Annotation
코드를 직관적이게 만들어 준다.
@Retention(RetentionPolicy.RUNTIME) // 컴파일 이후에도 JVM에 의해서 참조가 가능합니다.
@Retention(RetentionPolicy.CLASS) // 컴파일러가 클래스를 참조할 때까지 유효합니다.
@Retention(RetentionPolicy.SOURCE) // 어노테이션 정보는 컴파일 이후 없어집니다.
@IntDef를 enum같이 사용 가능하다. IntDef를 이용해 사용할 static 변수들을 추가 한 후 @intergace를
이용해 인터페이스 이름을 추가해 준다(여기선 LocationStatus). 그 뒤 @LocationStatus를 이용하여 이 변수들을 사용함을 알릴 수 있다.
~~~
    private static final int INDEX_WEATHER_ID = 0;
    private static final int INDEX_MAX_TEMP = 1;
    private static final int INDEX_MIN_TEMP = 2;
    private static final int INDEX_SHORT_DESC = 3;

    @Retention(RetentionPolicy.SOURCE)
    @IntDef({LOCATION_STATUS_OK, LOCATION_STATUS_SERVER_DOWN, LOCATION_STATUS_SERVER_INVALID,  LOCATION_STATUS_UNKNOWN})
    public @interface LocationStatus {} // 다음에 @LocationStatus로 이 자료들을 사용한다는 것을 알릴 수 있다.
~~~
### app Debugging
control + f8 을 누르면 중단점을 만들 수 있고
거기서 변수, 계산식등을 찾아볼 수 있다.

### Custom View 만들기
CustomView Attributes를 만들기 위해res/vlues 에 다음과 같이 선언한다.
~~~
    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <declare-styleable name="LocationEditTextPreference">
            <attr name="minLength" format="integer"/>
        </declare-styleable>
    </resources>
~~~

다음은 viewPreference를 상속받은 클래스를 만들어서 이 styleable을 사용한다. 참고 - https://developer.android.com/training/custom-views/create-view.html#applyattr
그 후 원하는 레이아웃에 CustomView를 넣고 정했던 attribute (예로 custom:minLength="3")을 넣어준다. 이 때 custom 을 익식하지 못하므로

    xmlns:custom="http://schemas.android.com/apk/res/com.example.android.sunshine.app" 와 같은 코드를 추가해 준다.
아직 attribute를 만들긴 했지만 아직 minLength가 뜻을 가진 것은 아니다.

### 글자수에 따라 버튼 활성화 하기
만들었던 preference view의 showDialog()를 오버라이드 하자(EditTextPreference의 경우 edit을 바꾸기 위해 클릭하면 dialog가 뜬다. 그 때 show dialog가 실행된다.) 여기에 getEditText()로 editText를 받은 뒤 addTextChangeListener로 리스너를 달아준다. new TextWatcher를 사용하며 afterTextChanged 에 getDialog()로 대화상자를 받은 뒤 instanceof로 AlertDialog인지 확인한다(EditTextPreference를 눌렀을 때 뜨는 dialog 종류가 AlertDialog 인듯 하다) 그 후 버튼을 찾아 글자 수에 따라 활성화, 비활성화 해 준다.
~~~
    @Override
    public void afterTextChanged(Editable editable) {
        Dialog dialog = getDialog();
        if( dialog instanceof AlertDialog){
            AlertDialog alertDialog = (AlertDialog)dialog;
            Button positiveButton = ((AlertDialog) dialog).getButton(AlertDialog.BUTTON_POSITIVE);
            if(editable.length()<mMinLength){
                positiveButton.setEnabled(false);
            }else{
                positiveButton.setEnabled(true);
            }
        }
    }
~~~

# Lesson3

### 접근성 높이기
content descriptions // android:contentDescription 속성을 사용하거나 setContentDescription()메소드를 사용하여 등록 할 수 있다. 이는 talkBack 앱으로 시각적으로 불편한 사용자에게 음성으로 알려준다.
focus-based navigation // 터치패드만 아닌 다른 hardware, software로의 입력도 받을 수 있어야 한다.
no audio-only feedback // 소리로 피드백을 줄 때 소리 뿐 아니라 진동, 시각적인 피드백도 함꼐 주어야 한다.
https://developer.android.com/training/accessibility/testing.html

### accessiblility service
- 접근성 서비스를 만들면 내 서비스를 다른앱에서 가져다 쓸 수 있고 나의 앱에서 다른 서비스를 가져다 쓸 수도 있다.

### RtL로 전환하기
https://android-developers.googleblog.com/2013/03/native-rtl-support-in-android-42.html
- manifest의 <application> 태그안에 android:supportsRtl="true"속성을 추가해 준다.
- minSDK 가 17이상이면 left/right 속성을 start/end속성으로 바꾸면 되고 이하이면 start/end속성을 추가해 줘야 한다.

### string.xml이용하기
- https://developer.android.com/distribute/tools/localization-checklist.html#manage-strings
- 주석을 달아 어디서 어떻게 쓰이는 건지 명시 할 수 있고, <xliff:g>태그를 이용해 번역하면 안된다는 것을 나타 낼 수도 있다.
- util을 이용해 날짜나 숫자 화폐 등등의 형식을 지역에 맞게 변경 시킬 수 있다.
- values/, values-en/ ,values-fr/ 등 여러 지역에 해당하는 value폴더를 만들 수 있고 후에 그 지역 폴더가 우선권을 가진다.

# Lesson4

### Glide 사용하기
이미지 로딩을 더 편하게 해 주고 이미지가 없을 경우 대체 이미지를 사용 하는 등 여러 기능이 있는 라이브러리이다.

##### Gradle 추가
compile 'com.github.bumptech.glide:glide:3.5.2'
##### 어떻게 작동하는지 확인 해 보자. 다음은 기존 setImageResource 를 Glide를 이용해서 하는 방법이다.
.load는 이미지를 url로 부터 가져오고 error는 만약 이미지가 없을 때 대체 이미지를 가져온다. into는 가져온 이미지를 imageview에 적용시킨다.
~~~
    //mIconView.setImageResource(Utility.getArtResourceForWeatherCondition(weatherId));

    Glide.with(this)
        .load(Utility.getArtUrlForWeatherCondition(getActivity(), weatherId))//Url에서 이미지를 불러오는 로컬 메소드이다.
        .error(Utility.getArtResourceForWeatherCondition(weatherId))//local에서 이미지를 불러오는 로컬 메소드이다.
        .into(mIconView);//imageview에 적용시킨다.
~~~
##### Glide로 이미지를 가져와서 largeIcon사이즈로 재 조정하기.
~~~
    //Bitmap largeIcon = BitmapFactory.decodeResource(resources, Utility.getArtResourceForWeatherCondition(weatherId));

    Bitmap largeIcon = Glide.with(context)
    .load(artUrl)
    .asBitmap() // 단순 bitmap 형태로 이미지를 가져온다.
    .error(artResourceId)
    .into(largeIconWidth, largeIconHeight)// fix width, height값을 지정해 준다.
    .get();
~~~
largeIcon의 크기는 진저브레드 이하에선 48dp 고정, 그 외에는 지정된 largeIcon 값을 사용한다.
~~~
    int largeIconWidth = Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB
    ? resources.getDimentionPixelSize(android.R.dimen.notification_large_icon_width)
    : resources.getDimensionPixelSize(R.dimen.notification_large_icon_default);
~~~
# Lesson5
##### Polling 과 pushing
Polling 은 device가 정기적으로 서버에 변화가 있는지 없는지 확인하는 것.
Pushing은 서버가 변화가 있으면 device를 call하는것
pushing에는 두가지 방법이 있는데 한가지는 tickle이라는 작은 메세지를 보내는 것이고 두번째는 업데이트 내용을 보내고 중간단계를 없애는 것이다.
pushing을 사용하면 sync와 인터넷도 적게 사용할 것이다.
##### GCM사용하기 (Firebase Cloud Messaging)
Google Cloud Messaging의 새 버전인 Firebase Cloud Massaging을 이용하게 바뀌었다... 자세한 방법은 http://blog.naver.com/PostView.nhn?blogId=cosmosjs&logNo=220739141098&categoryNo=0&parentCategoryNo=56&viewDate=&currentPage=1&postListTopCurrentPage=1&from=search 참고
##### Manifest추가하기
<uses-permission android:name="android.permission.IINTERNET"/>추가하기
<uses-permission android:name="android.permission.WAKELOCK"/>추가하기 - 메세지를 받았을 때 계속 슬립시킬건지 정할 수 있다. 여기선 활용화
<uses-permission android:name="com.google.android.c2dm.permission.RECEIVER"/> 메세지 받기 활성화
<permission adnroid:name="com.example.android.sunshine.app.permission.C2D_MESSAGE" android:protectionLevel="signature"/>
<uses-permission android:name="com.example.android.sunshine.app.permission.C2D_MESSAGE"/> 다른앱이 우리에게 오는 메세지를 받을 수 없게 한다.
그 외에 네개의 클레스를 추가 해야 한다.https://classroom.udacity.com/courses/ud855/lessons/3995738628/concepts/59769187270923# 참조
##### Registeration
서비스를 등록해야 한다. https://classroom.udacity.com/courses/ud855/lessons/3995738628/concepts/59769187290923# 참조
# Lesson6
##### Material Design
참고 http://kr.besuccess.com/2015/08/material-design/
메테리얼 디자인은 구글에서 내놓은 디자인 가이드라인이다.
##### Color
색과 빈공간을 더 효율적으로 사용하자
색상을 선택할 때 메테리얼 디자인을 위해 색상팔레트를 제공한다
100은 밝은 700~800은 어두운 500은 적당. A000는 강조색상
##### ActionBar to Toolbar
5.0은 새로운 툴바 위젯이 도입되었다. 하이라키가 나머지 색조와 잘 어우러지고 스크롤에 반응한다.요즘은 두개이상의 툴바를 사용한다.
스타일을 @style/Theme.AppCompat.Light.NoActionBar 로 바꾸어야 한다.
라이트 팝업테마를 반드시 사용해야 한다.
기본 바탕은 투명이다.
그 뒤 레이아웃에 툴바를 추가한다.
setSupportActionBar를 이용하여 툴바를 액션바처럼 이용하게 액티비티를 설정 해 준다.
getSupportActionBar로 후에 가져와 속성을 바꿀 수 있다.
툴바안에 ImageView를 넣어서 로고를 넣을 수 있는데 유용하다.
##### DetailView
메티리얼 디자인에 따라 세로로 16dp, 가로로 32dp의 공간이 추가 되어 잇다.
그리드 레이아웃을 사용하면 가로 크기에 따라 자동조정 되고 각 뷰에서 제약 조건을걸 수도 있다(자체 layout_gravity를 이용한다).
열의 개수를 columnCount 속성을 이용해 설정할 수 있다(자동으로 넣는다).
순서대로 잇을 때 그 중 한 아이콘의 위치를 바꾸면 그 뒤에 있는 view들은 자동으로 그 뒤에 정렬된다.
한 view를 두 열이 차지하도록 변경 할 수도 있다. weight를 이용해 벨런스를 잡자.

##### GridView 사용하기
설정을 위해 엡에 GridLayout 특정 XML속성을 정의하고 res-auto scheme를 추가한다.https://classroom.udacity.com/courses/ud855/lessons/3940839262/concepts/43340102820923#
빈 공간을 넣을 때는 space 뷰를 사용하자
ImageView 의 adjustViewBounds를 이용하면 이미지의 크기를 제한 할 수 있다. tools:src속성을 이용하여 에디터 도구에서 이미지를 보여준다.
##### 그림자 그리기
layout Language에 elevation이라는 파라미터를 사용 할 수 있다. z값을 주어 높게주면 더 크고 부드러운 그림자를 가진다. 뒤의 뷰를 가릴 수 있다.
Elevation android:elevation=@dimen/appbar_elevation"
##### 작은화면 -> 큰화면
큰화면으로 바꿀때 내용크기가 같다면 좌우 공간의 크기가 커져야 한다. values/sw600dp-port/dimens.xml 에 extra-padding을 정해 놓자. 이는 가로가 600dp이상일 때 참고한다는 뜻이다. 속성은 android:paddingLeft="@dimen/extra-padding" 같이 이용하면 된다.
##### 같은 화면 재사용 할 때
<include>를 이용하자.
##### CardView
elevation이 잘 안먹힐 때가 있다. CardView를 이용하자
##### Scrolling in Parallel
HONEYCOMB 이상이라면 Recycler view에 scroll listener를 추가해 준다. @TargetApp(Build.VERSION_CODES.HONEYCOMB)이라는 어노테이션을 쓰면 된다. x,y 변화량을 반환한다 -> 그에따라 반응하게 만든다.
##### 이동 애니메이션화
어떻게 움직일지를 정하는<transitionSet>을 담는 transition resource를 만들어야 한다.
~~~
<fade>
    <targets>
        <target android:excludeId>//excludeId는 이 아이디해당 빼고 모든 뷰 포함.
    <targets>
<fade>
이런식으로 어떤걸 사라지게 할지 타겟을 정할 수 있다.
~~~    
후에 appTheme에 추가하여 실행하자.
##### Shared element transition
서로다른 엑티비티에서 공유하는 view를 자연스럽게 변하도록 해 주는 방법
두 activity에 어떤 view가 공통되는지 명시한다. 두 layout에 transitionName을 추가한다.
그리고 transition 시작점에 있는 view들에게도 setTransitionName을 이용해 이름을 추가한다.
supportPostponeEnterTransition을 이용하여 다른 layout이 완료될 때 까지 기다릴 수 있다.
후에 supportStartPostponeEnterTransition()이 준비가 끝나면 알려준다.
##### Scrolling the AppBar
AppBar가 어떻게 움직이는지 배운다.
CoordinatorLayout
기본적으로 frameLayout과 비슷하다. 내부 view는 gravity로 조정가능하다. parentView에 따라 위아래좌우에 배치될 수 있다.
layout을 변경할 수 있는 behavior classes를 지원한다.
AppBarLayout은 특별한 종류의 linear layout으로 CoordinatorLayout과 같이 동작하여 다양한 종류의 toolbar 패턴을 그 밑의 CoordinatorLayout에서 스크롤 하는동안 형태를 바꾼다. 이때 AppBar는 어떤 행동을 할지 layout scroll flag를 입력해둬야 한다. 아래에 있는 CoordinatorLayout은 그 동안 AppBar Scrolling view Behavior를 받는다(밑에 레이아웃도 어떻게 움직여야하는지 알아야 하는듯)
AppBar에 Collapsing ToolbarLayout을 추가 할 수 있다.(스크롤 내리면 먼저 사라지는 부분)
가장 좋은점은 XML layout만 이용하고 코드를 건들지 않아도 된다.
##### Adding Elevation
Appbar가 움직일 때 그림자를 드리우자
RecyclerView에 스크롤리스너 onScrollListener를 달 수 있다.
그리고 appbar.setElevation()을 이용하여 엘레베이션을 달 수 있다.
# Lesson8
##### PlaceAPI
사용하기 위해 먼저 API Key를 받고 매니페스트에 등록 해 준다.https://console.developers.google.com/flows/enableapi?apiid=placesandroid&keyType=CLIENT_SIDE_ANDROID
FCM을 onActivity 시점에서 sdk사용이 가능한지 확인하고 불가능하다면 placePicker을 실행하는 위젯을 만들고 불가능할 경우 만들지 않는다.
현재표시를 나타내는 이미지를 사용하는데 이미지 아이콘은 작다. 하지만 그걸 감싸는 frameLayout의 크기는 크다. 그래서 frameLayout에 클릭리스너를 달자
##### placeAPI 사용
gradle 추가
compile 'com.google.android.gms:play-services-location:7.5.0'
Manifest추가
<uses-permission adnroid:name="android.permission.ACCESS_FINE_LOCATION"/>
onActivityResult에서 원하는 intent인지 확인하고 preference를 바꾼다.
받은 정보가 주소인지 좌표인지 알 수가 없다. 하지만 위도, 경도를 꼭 가지고 있기 때문에 이를 이용한다.
