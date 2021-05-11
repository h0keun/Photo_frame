### Photo_frame

+ Android Premission(load image)
+ View Animation
+ Activity Lifecycle
+ Content Provider(SAF : Storage Access Framework)

### 2021.05.11

+ layout.xml
  - ConstraintLayout속성중에 ```app:layout_constraintDimensionRatio="3:1"``` 를 이용해서  
    LinearLayout 화면비율을 세로 1 : 가로 3 으로 나누고 그안에 들어가는 3개의 View에 weight를 1씩 주어 깔끔하게 영역분할
  - ConstraintLayout안에 ImageView 2개를 서로 곂쳐서 자연스러운 fading 효과(클래스단에서 설정해주어야함) 를 주기위한 화면구성
  - ImageView.ScaleType [공식](https://developer.android.com/reference/android/widget/ImageView.ScaleType) , [예시](https://parkho79.tistory.com/71)

+ Kotlin.class
  - apply, with, let, also, run(따로 정리)
  - 앱 권한요청 👉 [공식](https://developer.android.com/training/permissions/requesting?hl=ko)
  - 갤러리에서 이미지 가져오기
    ```KOTLIN
    private fun navigatePhotos() {
        val intent = Intent(Intent.ACTION_GET_CONTENT)
        intent.type = "image/*"
        startActivityForResult(intent, 2000)
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

+ 체크할 문법
  - .apply / ?.let / .forEachIndexed


💡 URI vs URL vs URN
