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
