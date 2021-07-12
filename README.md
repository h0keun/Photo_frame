### Photo_frame

+ Android Premission(load image) : 액티비티에서 외부저장소 접근
+ View Animation : 뷰 에니메이터를 이용. fade-in/out 처럼 보이도록 구현
+ Activity Lifecycle
+ Content Provider(SAF : Storage Access Framework) : 스마트폰 저장소의 사진선택

<img src="https://user-images.githubusercontent.com/63087903/125246426-bd5d3300-e32c-11eb-9787-23b4ca72eda3.jpg" width="200" height="430"> <img src="https://user-images.githubusercontent.com/63087903/125246425-bcc49c80-e32c-11eb-8df0-f2e228ff9450.jpg" width="200" height="430"> <img src="https://user-images.githubusercontent.com/63087903/125246423-bb936f80-e32c-11eb-8ecc-ebe6a418c026.jpg" width="200" height="430">

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
+ 권한 받아오기(MainActivity.kt 코드 참조)
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
  
  * 초기에 
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

#### Kotlin.class (MainActivity.kt / PhotoFrameActivity.kt)
+ MainActivity - 갤러리에서 이미지 가져오기
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
    ```
     개선점 : startActivity에 들어가는 intent를 createChooser를 이용하여 사용자에게 선택권을 주는 것이 좋다.  
     단순히 기본 갤러리앱에서만 사진을 가져오는것이 아니라 구글 드라이브, 기타 갤러리앱 등 사용자에게 선택권을 주고  
     개발자는 그에 대한 Uri값을 얻어 Path로 변환, 추출하여 업로드 등 작업을 수행하면 된다.
     
  - 갤러리에서 가져온 이미지 다른 액티비티에서 띄우기
    ```KOTLIN
    // MainActivity.kt
    
    private fun initStartPhotoFrameModeButton() {
        startPhotoFrameModeButton.setOnClickListener {
            val intent = Intent(this, PhotoFrameActivity::class.java)
            imageUriList.forEachIndexed { index, uri ->
                intent.putExtra("photo$index", uri.toString())
            }
            intent.putExtra("photoListSize", imageUriList.size)
            startActivity(intent)
        }

    }
    
    // PhotoFrameActivity.kt
    
    private fun getPhotoUriFromIntent() {
        val size = intent.getIntExtra("photoListSize", 0)
        for (i in 0..size) {
            intent.getStringExtra("photo$i")?.let {
                photoList.add(Uri.parse(it))
            }
        }
    }
    ```
  - 최종적으로 timer 기능을 통해 딜레이를 부여하고 runOnUiThread를 통해 이미지를 뿌린다.(전자액자구현)
    ```KOTLIN
    private fun startTimer() {
        timer = timer(period = 5 * 1000) {
            runOnUiThread {

                Log.d("PhotoFrame", "5초가 지나감 !!")

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
    ```
    이 때 안드로이드 생명주기에 맞추어서 타이머를 켜거나 꺼줘야 하는데  
    ``` onStop() -> timer?.cancel() 을 onStart() -> startTimer() 를 onDestroy() -> timer?.cancel()```


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
💡 URI vs URL vs URN  
💡 Intent.ACTION 에는 어떤것들이 존재할까?  
💡 ContentProvider - uri 심화적으로 알아보기
💡 Matisse 라이브러리를 통해 갤러리연동해서 해보기!
