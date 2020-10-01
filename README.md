# FSRCNN : https://arxiv.org/abs/1608.00367

#### <I studied while referring to various sites, but it is not enough. Thanks anytime for feedback.>
### <heejo5@naver.com>

Accelerating the Super-Resolution Convolutional Neural Network
--------------------------------------------------------------
* CNN을 SR에 접목시켜 큰 관심을 가진 SRCNN은 좁은 IMG영역에 대한 정보만을 사용하고 낮은 lr로 인해 학습을 하는데 며칠이 걸리고 단일 Scale Img에 대해서만 가능하다는 제약이 있었음
* 속도가 매우느리다는점을 개선하고 나온 FSRCNN을 보게되면 FSRCNN은 MxN의 LR Img로 2Mx2N의 bicubic을 만들어 내는 과정을 생략 
* Network 마지막에 Deconvolution Layer를 두었고 기존 SRCNN에서 Network를 크게 만들수록 Parameter 의 수가 많아지는 것을 방지하고자 인코더 디코더 구성과 비슷한 Shrinking –Expanding 모래시계 모양 구조로 변경하여 Parameter 수를 줄였으며 SRCNN 과 비슷한 성능은 유지되지만 속도는 더 빨라졌음
* 여러 Scaling을 지원하기 위해 마지막 Deconvolution Layer만 Fine tuning하면됨

FSRNN Architecture
------------------
![image](https://user-images.githubusercontent.com/61686244/94804135-4867df00-0425-11eb-8e27-89c2066855c9.png)
* FSRCNN은 기존 SRCNN의 3단계 과정을 5단계의 과정으로 변형
* Expanding, Deconvolution 5단계의 과정을 통해 LR을 HR로 만들어주게됨
* Feature Extraction 과정은 입력 영상으로부터 LR특징을 추출하는 단계로 5x5 크기의 커널로 56개 Feature Map을 만들게되고 Encoding 기능을하는 Shrinking과정은 56개의 LR의 Feature Map을 12개의 Feature Map으로 압축하는 단계로 1x1 Conv를 사용하여 Mapping 과정의 연산 부하를 줄였음
* Mapping과정으로 LR 특징으로부터 HR특징을 추론하는 단계로 3x3 CONV 4개의 층으로 구성 되어 있음
* Decoding 기능을 하는 Expanding과정으로는 12개의 Feature Map을 56개로 압축해제 하는 단계로 1x1 Conv를 사용
* 마지막 HR Img를 뽑아내기 위한 과정으로 Deconvolution 단계로 9x9 Conv 층으로 구성 되어 있음
* FSRCNN 은 Mapping단계에서 Feature Map의 크기를 12로 줄여 SRCNN 보다 빠른  SISR을 구현하였고 신경망의 깊이를 SRCNN 3개 층에서 8개의 층으로 늘려 복원 성능도 향상 

Shrinking, Expanding 과정에서 1x1 Conv를 사용하는 이유
----------------------------------------------------
![image](https://user-images.githubusercontent.com/61686244/94804392-b57b7480-0425-11eb-85d6-df4ba360dee1.png)
* 기존 SRCNN에서 사용하는 5x5 Conv를 그대로 사용하게 된다면 위에 보이시는 예제처럼 총 작업수가 112.9M가 나오게됨
* 오른쪽 처럼 1x1 Conv를 적용한 과정을 추가해서 보게된다면 5.3M로 파라메터의 수가 확실하게 줄어드는 것을 확인 할 수 있음
* 1x1 Conv 과정을 Shrinking와 Expanding과정에 각각 적용을하여 파라메터의 수를 줄여 네트워크 작업 속도를 올림

Mapping 과정에서 멀티의 3x3 Conv를 사용하는 이유
----------------------------------------------
![image](https://user-images.githubusercontent.com/61686244/94804619-130fc100-0426-11eb-87c3-8cba2922d393.png)
* 5x5 Conv를 사용하게 되면 25개의 파라미터를 사용하게 되지만 3x3 Conv 2개를 사용하게 된다면 총 18개의 파라미터만 필요하게 됨으로 28%가 감소하는 효과를 볼 수 있음
* 멀티의 Conv를 사용하여 파라메터의 수를 줄여 네트워크의 속도를 높혔음

The transions from SRCNN to FSRCNN
----------------------------------
![image](https://user-images.githubusercontent.com/61686244/94804754-536f3f00-0426-11eb-87b4-1ca1e4fe8b3d.png)

다양한 Scale에 따른 FSRCNN 전략
------------------------------
![image](https://user-images.githubusercontent.com/61686244/94804881-7e599300-0426-11eb-8e57-0f94c9651a32.png)
* FSRCNN의 이점 중 하나로 사전에 학습된 FSRCNN을 이용하여 다른 upscale factor 또한 빠르게 학습을 할 수 있음
* Up scale 에 대한 정보를 포함하고 있는 De convolution 층과 LR img에서 추출한 convolution 층으로 이루어져 있는데 실험을 통해서 밝혀낸 결과 convolution 층은 다른 up scale factor에 대해서도 동일하게 feature 추출 기로 사용을 할 수 있음
* 이러한 특성을 이용하여 deconvolution층을 미리 다양한 upscale fatcor에 대해서 학습을 해 놓고 각각의 결과를 볼 때 사전 학습된 deconvolution 층을 이용하여 더 빠른 결과를 나오게 만들었다고함

The comparison of PSNR
----------------------
![image](https://user-images.githubusercontent.com/61686244/94805053-b95bc680-0426-11eb-872f-e2e486758711.png)
* 다른 셋팅환경에서 PNSR 값을 비교해본 결과로서 Network의 Mapping의 깊이는 2개 3개 4개 그리고 LR 특징 , shrinking 필터의 수에 따라 각각 비교해본 결과
* 파라미터수와 함께 비교해 보았을때 가장 좋은 성능은 FSRCNN (56,12,4)에서 발견할 수 있었고 또한 이미 FSRCNN(48,12,2)에서 이미 SRCNN보다 더 높은 결과값에 도달한 것을 확인 할 수 있음

The results of PSNR and test time
---------------------------------
![image](https://user-images.githubusercontent.com/61686244/94805209-f7f18100-0426-11eb-8242-17339cda3a76.png)
![image](https://user-images.githubusercontent.com/61686244/94805235-0344ac80-0427-11eb-9786-dbaa8aca4d10.png)
![image](https://user-images.githubusercontent.com/61686244/94805253-0cce1480-0427-11eb-9693-19562da683e2.png)

Conclusions
-----------
* 기존 SRCNN과 다르게 인코더-디코더 구조와 비슷한 5개의 구조로 변경하여 속도를 빠르게 개선한 FSRCNN
* 마지막 Layer에 Deconvolution Layer를 둠
* SRCNN과 다르게 Interpolation없이 LR에서 바로 mapping을 함
* Expanding하기 전에 Input 차원을 줄임
* 필터크기를 줄이고 Mapping Layer를 늘림
* CPU로 24fps 이상 뽑아냄
