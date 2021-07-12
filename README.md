### Photo_frame

+ Android Premission(load image) : 액티비티에서 외부저장소 접근
+ View Animation : 뷰 에니메이터를 이용. fade-in/out 처럼 보이도록 구현
+ Activity Lifecycle
+ Content Provider(SAF : Storage Access Framework) : 스마트폰 저장소의 사진선택

<img src="https://user-images.githubusercontent.com/63087903/125246426-bd5d3300-e32c-11eb-9787-23b4ca72eda3.jpg" width="200" height="430"> <img src="https://user-images.githubusercontent.com/63087903/125246425-bcc49c80-e32c-11eb-8df0-f2e228ff9450.jpg" width="200" height="430"> <img src="https://user-images.githubusercontent.com/63087903/125246423-bb936f80-e32c-11eb-8ecc-ebe6a418c026.jpg" width="200" height="430"> <img src="https://user-images.githubusercontent.com/63087903/125254330-52fcc080-e335-11eb-9270-efb5fb124167.jpg" width="200" height="430">

### [2021-05-11]
### [2021-07-12]

#### manifest 
+ 유저 퍼미션, 액티비티 추가
  ```KOTLIN
  * 외부 스토리지 읽기권한 추가
  <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

  * 슬라이드 포토를 보여줄 액티비티 추가
  <activity android:name=".PhotoFrameActivity"
    android:screenOrientation="landscape"/>
   
  // 보통 많이 사용되는 orientation값은
  // Portrait(세로모드 유지)
  // landscape(가로모드 유지)
  // fullSensor(기기의 sensor에 다라서 방향이 결정)
  // 따로 설정해주지 않으면 default 값으로 unspecified 가 적용되고 이는 시스템에 설정된 값에 따라 화면방향이 결정된다.
  ```
#### layout.xml (activity_main.xml / activity_photoframe.xml)
+ acivity_main.xml
  ```KOTLIN  
  <androidx.constraintlayout.widget.ConstraintLayout
    ...
    <LinearLayout
      ...
      app:layout_constraintDimensionRatio="3:1"
      ... >
        <ImageView
          ...
          android:scaleType="centerCrop"
          android:layout_weight="1"/>
        
        <ImageView
          ...
          android:scaleType="centerCrop"
          android:layout_weight="1"/>
        
        <ImageView
          ...
          android:scaleType="centerCrop"
          android:layout_weight="1"/>
      </LinearLayout>
      
      ... 6개의 칸을 만들기위해 위의 LinearLayout을 반복
      
  </ConstraintLayout>
  
  * ConstraintLayout속성을 이용해 영역을 분할하여 LinearLayout에 3개의 ImageView를 배치 
  * LinearLayout 화면비율을 세로 1 : 가로 3 으로 나누고,
    그 안에 들어가는 3개의 ImageView에 weight를 1씩 주어 깔끔하게 영역분할
  ```
+ activity_photoframe.xml
  ```KOTLIN
  * ConstraintLayout영역을 전부 차지하는 ImageView 2개를 서로 곂쳐서 화면 전체에 Image가 보이도록 함
    ImageView를 겹친 이유는 자연스러운 ViewAnimation 효과(클래스단에서 설정해주어야함) 를 주기위한 화면구성이다.
  ```
+ ImageView.ScaleType [공식](https://developer.android.com/reference/android/widget/ImageView.ScaleType) , [예시](https://parkho79.tistory.com/71)

#### Kotlin.class (MainActivity.kt)
+ 권한 받아오기 - 사진추가하기 버튼 클릭시 popup창을 띄워서 유저에게 권한요청을 한다.
  ```KOTLIN
  private fun initAddPhotoButton() {
      addPhotoButton.setOnClickListener {
          when {
              ContextCompat.checkSelfPermission(
                      this,
                      android.Manifest.permission.READ_EXTERNAL_STORAGE
              ) == PackageManager.PERMISSION_GRANTED -> {
                  navigatePhotos()
              }
              shouldShowRequestPermissionRationale(android.Manifest.permission.READ_EXTERNAL_STORAGE) -> {
                  showPermissionContextPopup()
              }
              else -> {
                  requestPermissions(arrayOf(android.Manifest.permission.READ_EXTERNAL_STORAGE), 1000)
              }
          }
      }
  }
  
  * 사진추가버튼을 누르면 when 문을 통해 순차적으로 검사하게된다.
    1. ContextCompat.checkSellfPermission() 메서드를 통해 앱에 이미 권한이 부여되었는지 확인 
    2. 권한이 수락된 상태라면( == PackageManager.PERMISSION_GRANTED)
    3. navigatePhotos() 를 실행시킨다. ( SAF 기능을 사용해 사진 uri를 받아오는 함수)
    
    4. shouldShowRequestPermissionRationale 권한 수락이 거절된 상태라면 호출하는 메서드.
       왜 권한이 필요한지에 대한 교육용 팝업을 띄운다. 
       (showPermissionContextPopup() : 첨부한 사진 4번째)
    5. requestPermissions (권한을 요청하는 팝업 : 첨부한 사진 1번째)
       (시스템이 권한 요청 코드를 관리하도록 허용하는 대신 권한 요청코드를 직접 관리할 수 있다. 
       🥕 사용자가 시스템 권한 대화상자에 응답하면 시스템은 앱의 onRequestPermissionsResult()구현을 호출하게 된다.)

  * 부연설명하자면 초기에 사진추가버튼을 누르면 애초에 권한에대한 수락/거부 여부가 없기 때문에
    else문 안의 5 팝업을 띄우게되고
    여기서 만약 권한을 허용했다면 버튼 클릭시 1,2,3 을 실행하는것이고,
    권한을 거부했다면 버튼 클릭시 4 를 실행하여 권한이 필요한 이유에대한 설명을 보여주게되는것이다.
  ```
+ PopUp() 두가지 (교육용 팝업 / 권한요청 팝업)
  ```KOTLIN
  * 교육용 팝업 : showPermissionContextPopup() : 권한요청 거절당했을 시 실행
               
  private fun showPermissionContextPopup() {
      AlertDialog.Builder(this)
              .setTitle("권한이 필요합니다.")
              .setMessage("전자액자에 앱에서 사진을 불러오기 위해 권한이 필요합니다.")
              .setPositiveButton("동의하기") { _, _ ->
                  requestPermissions(arrayOf(android.Manifest.permission.READ_EXTERNAL_STORAGE), 1000)
              }
              .setNegativeButton("취소하기") { _, _ -> }
              .create()
              .show()
  }
  
  * 권한요청 팝업 : requestPermissions(arrayOf(android.Manifest.permission.READ_EXTERNAL_STORAGE), 1000) 
                   해당 메서드에서 사용자가 시스템 권한 대화상자에 응답하면 시스템은 앱의 onRequestPermissionsResult()구현을 호출!
                   
  override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<out String>, grantResults: IntArray) {
      super.onRequestPermissionsResult(requestCode, permissions, grantResults)

      when (requestCode) {
          1000 -> {
              if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                  navigatePhotos()

              } else {
                  Toast.makeText(this, "권한을 거부하셨습니다.", Toast.LENGTH_SHORT).show()
              }
          }
          else -> {
               //
          }
      }
  }

  // 🥕 임의로 지정한 requestCode = 1000 의 값을 비교하여 실행시킴! 여러가지 권한요청이 있을 수 있고, 
  // 🥕 사용자가 원하는 권한만 부여하는게 가능하도록 지금까지의 내용처럼 requestCode를 통해 구분짓고, 배열에 담아서 확인이 가능하다!
  ```
+ MainActivity - 갤러리에서 이미지 가져오기(SAF)
  ```KOTLIN
  private fun navigatePhotos() {
      val intent = Intent(Intent.ACTION_GET_CONTENT)
      intent.type = "image/*"              // image 파일 형식 받아오기
      startActivityForResult(intent, 2000) // requestCode: 2000으로 구분짓기
  }
    
  ...

  override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
      super.onActivityResult(requestCode, resultCode, data)

      if (resultCode != Activity.RESULT_OK) {
          return
      }

      when (requestCode) {
          2000 -> {
              val selectedImageUri: Uri? = data?.data

              if (selectedImageUri != null) {

                  if (imageUriList.size == 6) {
                      Toast.makeText(this, "이미 사진이 꽉 찼습니다.", Toast.LENGTH_SHORT).show()
                      return
                  }

                  imageUriList.add(selectedImageUri)
                  imageViewList[imageUriList.size - 1].setImageURI(selectedImageUri)

              } else {
                  Toast.makeText(this, "사진을 가져오지 못했습니다.", Toast.LENGTH_SHORT).show()
              }


          }
          else -> {
              Toast.makeText(this, "사진을 가져오지 못했습니다.", Toast.LENGTH_SHORT).show()
          }
      }
  }
  
  * navigatePhotos() 에서 Intent.ACTION_GET_CONTENT 로 SAF 기능을 실행시켜서 컨텐츠를 가져오는
    안드로이드 내장 액티비티를 실행시키고 모든 이미지타입들만 설정해 필터링 한후에
    선택된 컨텐츠를 startActivityForResult(intent, 2000)으로 requestCode = 2000 으로 받아온다.
    
  * startActivity()의 짝인 onActivityResult()를 오버라이드하여 requestCode = 2000 인 컨텐츠를 받아오며 
    resultCode 를 통해 사진을 선택한 경우(Activity.RESULT_OK)와 
    그냥 캔슬한 경우(Activity.RESULT_CANCLE // 여기서는 그냥 else로 처리) 로 나누어 모두 처리해 주었다.
  
  * onActivityResult를 사용한 이유는 바로 SAF를 통해 사용자가 내놓은 결과(사진)를 그대로 받아서 처리해주기 위함이다.
  ```
     개선점 : startActivity에 들어가는 intent를 createChooser를 이용하여 사용자에게 선택권을 주는 것이 좋다.  
     단순히 기본 갤러리앱에서만 사진을 가져오는것이 아니라 구글 드라이브, 기타 갤러리앱 등 사용자에게 선택권을 주고  
     개발자는 그에 대한 Uri값을 얻어 Path로 변환, 추출하여 업로드 등 작업을 수행하면 된다.
     
#### kotlin.class (PhotoFrameActivity.kt)
+ 갤러리에서 가져온 이미지 다른 액티비티에 받아오기
  ```KOTLIN
  * MainActivity.kt 
    : 전자액자 실행하기 버튼 클릭시 intent를 통해 이미지 uri 리스트들을 넘김(리스트 사이즈(이미지 갯수) 도 함께)
    
  private fun initStartPhotoFrameModeButton() {
      startPhotoFrameModeButton.setOnClickListener {
          val intent = Intent(this, PhotoFrameActivity::class.java)
          imageUriList.forEachIndexed { index, uri ->
              intent.putExtra("photo$index", uri.toString()) // 이미지 uri를 string으로 변환하여 담아줌
          }
          intent.putExtra("photoListSize", imageUriList.size)
          startActivity(intent)
      }
  }
    
  * PhotoFrameActivity.kt
    : 우리가 전달한 이미지 uri 를 꺼내준다.
    
  private fun getPhotoUriFromIntent() {
      val size = intent.getIntExtra("photoListSize", 0)
      for (i in 0..size) {
          intent.getStringExtra("photo$i")?.let {
              photoList.add(Uri.parse(it)) // string으로 변환해 보냈던 uri를 다시 이미지 Uri로 파싱
          }
      }
  }
  ```
+ 최종적으로 timer 기능을 통해 딜레이를 부여하고 runOnUiThread를 통해 이미지를 뿌린다.(전자액자구현 & 생명주기 맞추기)
  ```KOTLIN
  private fun startTimer() {
      timer = timer(period = 5 * 1000) { // 5초마다 작동하는 timer
          runOnUiThread {
              val current = currentPosition
              val next = if (photoList.size <= currentPosition + 1) 0 else currentPosition + 1

              backgroundPhotoImageView.setImageURI(photoList[current])

              photoImageView.alpha = 0f
              photoImageView.setImageURI(photoList[next])
              photoImageView.animate()
                      .alpha(1.0f)
                      .setDuration(1000)
                      .start()

              currentPosition = next
          }
      }
  }
  
  * xml에서 ImageView를 겹친이유는 Fade in/out 효과를 주기 위함이다.
    위와같이 리스트로 담긴 이미지가 있을 때 current와 next 이미지 둘다 화면에 띄우고
    alpha 값을 주어 투명도를 조절해서 1초동안 0f - 1.0f 로 전환되도록 
    animate()을 실행하면 자연스러운 전환이 가능하다.
  
  * 타이머를 onCreate()에서 실행하게되면, 
    다른 앱으로 전환하거나 전화가 오는 등의 상황에도 이 타이머가 꺼지지 않고 잔존하게 된다.
    이는 앱이 강제종료되는 상황으로 이어질 수 있으니 생명주기에 따라서 아래와 같이 타이머를 켜고 꺼주어야 한다.
  
    onStop() -> timer?.cancel()  
    onStart() -> startTimer()  
    onDestroy() -> timer?.cancel()

  * 액티비티 실행시 onCreate() > onStart() > onResume()을 거쳐 화면에 보여지고 유저와 상호작용하게 된다.
    이때, 다른 액티비티로 넘어간다거나, 홈키를 누르거나, 다른앱이 실행되는 등의 경우에  onPause()가 되어 액티비티는 일시정지가 되고,
    화면에서 보이지 않게되면 onStop()이 호출된다.
    이후 다시 앱을 실행하면 onResume()이 되어 다시 앱이 작동한다.
    
  * 액티비티가 onStop()상태 일때, 만약 메모리를 많이 차지하고 우선순위가 높은 다른 앱이 실행되었다면,
    App process Killed가 되고 그렇게되면 기존 앱 재실행 시에 onResume()이 아닌 onCreate()부터 다시 시작되게 된다.
    반대로 앱이 죽지 않았을 경우에 는 onRestart()를 거쳐 onStart()를 통해 실행된다.
    
  🥕 말로쓰자니 어수선한데 이는 구글에 안드로이드 생명주기를 검색해보면 다이어그램과 함께 
     상세하게 설명이 나와있으니 그것을 참조하도록 하자
  ```

💡 apply  
```KOTLIN
private val imageViewList: List<ImageView> by lazy {
    mutableListOf<ImageView>().apply {
        add(findViewById(R.id.imageView11))
        add(findViewById(R.id.imageView12))
        add(findViewById(R.id.imageView13))
        add(findViewById(R.id.imageView21))
        add(findViewById(R.id.imageView22))
        add(findViewById(R.id.imageView23))
    }
}

본 프로젝트에서는 apply를 위와같이 사용했는데, 기억하기로는 apply 없이
mutableList에 add를 하게되면 6개의 Image를 순차적으로 add해주는 것이라면
apply는 add하는 6개의 과정을 하나로 묶어 실행된 결과를
list에 한번에 담아주는것이라고 기억함

어떠한 연속적인 과정이 주어질 때 apply로 묶어서 효율을 높일 수 있다고 들었던것 같음
🥕 관련된 부분에 대해서는 조금더 알아보쟈
```
💡 권한요청 공식 [📌](https://developer.android.com/training/permissions/requesting?hl=ko)  
최대한 이해하기 쉽게 풀어서 설명했지만 그래도 부족한부분이 많다. 공식문서를 다시보자! 
💡 URI vs URL vs URN  
💡 Intent.ACTION 에는 어떤것들이 존재할까?  
💡 ContentProvider - uri 심화적으로 알아보기
💡 Matisse 라이브러리를 통해 갤러리연동해서 해보기!
