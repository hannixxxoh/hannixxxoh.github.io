---
title: "3DCrowdNet Code"
date: 2024-05-21 1:00:00 +0900
categories: [Code Review]
tags: [human, 3D, SMPL]
---


## 코드
이 글은 2024년 5월 21일 기준 3DCrowdNet Github()을 바탕으로 작성되었습니다.
https://github.com/hongsukchoi/3DCrowdNet_RELEASE

## 3DCrowdNet_RELEASE/demo/demo.py


### Load SMPL joint set

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/b7e3070b-15ba-4c5c-ab6b-939b0dbdf2d0)

SMPL joint의 개수, 이름, 좌우 대칭, 연결관계를 선언한다.



### Load MSCOCO joint set

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/ef01ef93-a03e-4bc8-8f5e-36f723f07e1f)

MSCOCO joint의 이름과 연결관계를 선언한다. 원래 MSCOCO joint는 총 17개지만 Pelvis와 Neck을 추가하였다. (엉덩이의 중간점과 어깨의 중간점으로 계산)

동일한 MSCOCO joint를 시각화를 위해 선언한다.




### Load checkpoint model

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/94b34faa-9e0a-46d3-ab7a-ece8441ae3e9)

모델의 checkpoint를 불러온다. get_model 함수는 3DCrowdNet_RELEASE/main/model.py에 다음처럼 선언되어 있다.

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/91aec407-0b7f-4875-80cc-77aa67af6187)

결과로 출력할 mesh의 vertex 개수와 joint 개수를 파라미터로 넘겨준다. (데이터셋 별로 다른 joint를 사용하기 때문이 아닐까 추측함)



### Process coco joint

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/e00914f9-d598-4645-b896-596e693c5c98)

논문에서는 이미 구현되어 있는 2D human pose estimation 모델로 joint를 추정하여 활용한다.(아마 데모 코드라서 json 파일에 미리 넣어 놓은듯?)

- **pose2d_result**: 여러 이미지의 각 사람의 coco joint가 저장되어 있는 딕셔너리
    - dict{'image_name': list(num of people x num of coco joints x (x, y, confidence, #, #), ...)}
    - #는 뭔지 모르겠음...

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/cd6afdad-52be-49e8-b9ba-47ad5bbbae5b)

데모용으로 넣어둔 이미지를 cv2의 imread 함수로 읽고, 이미지의 height와 width를 저장한다.

- **coco_joint_list**: 한 이미지의 각 사람의 coco joint가 저장되어 있는 리스트
    - list(num of people x num of coco joints x (x, y, confidence, #, #))

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/a12c608b-8dad-49c0-9236-f5b63221b713)

각 사람별 joint들을 coco_joint_img에 저장한 뒤, 원본 coco joint에는 없는 pelvis와 neck를 추가한다. <br>
confidence가 threshold(0.1)보다 높은지 여부를 coco_joint_valid에 저장한다. <br>
모든 joint의 confidence 합이 1보다 낮은 경우 필터링한다. (inaccurate input)
또한 이미 추가한 joint와 비교해서 완전히 똑같거나 차이값의 joint 평균이 20 이하인 경우 필터링한다. (same target) 

- **coco_joint_img**: 각 사람의 coco joint가 저장되어 있는 리스트
    - list(num of coco joints x (x, y, confidence, #, #))

- **drawn_joints**: 필터링된 사람의 coco joint가 저장되어 있는 리스트
    - list(num of filtered people x num of coco joints x (x, y, confidence, #, #))



### Prepare bbox

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/f5ac3700-48fc-4efb-b17d-eb33f3895399)

- get_bbox: coco joint를 통해서 사람의 bbox를 만드는 함수. 3DCrowdNet_RELEASE/common/utils/preprocessing.py에 다음처럼 정의되어 있다.
    - return 값: xmin, ymin, width, height (MSCOCO 형식이랑 똑같음)

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/5d6b1434-3a79-4c25-9b91-6da1da0b1e02)

- process_bbox: get_bbox에서 만든 bbox를 가지고 좌표가 이미지를 넘어가지 않도록 조정하는 함수. 3DCrowdNet_RELEASE/common/utils/preprocessing.py에 다음처럼 정의되어 있다.
    - return 값: xmin, ymin, width, height (이미지의 종횡비에 맞춰서 조정된 값)

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/b6446968-4b7d-4b9b-9970-8885fdb068ba)


![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/f3dd2677-f088-4f44-bde9-e6c7b76ad007)

정상적인 사람의 bbox는 모두 구했지만, bbox 크기가 다 다른 상태이다. 모델에 넣어주기 위해서는 일정한 사이즈로 사람의 bbox를 normalize해줄 필요가 있다. 이를 위해서 generate_patch_image 함수를 통해서 tranformation matrix와 inverse transformation를 구한다. (bbox -> cfg.input_img_shape) 

기본 config 파일에서 shape은 256 x 256으로 설정되어 있다. transform된 이미지를 Tensor로 변경하고 normalize한 뒤 cuda로 보낸다.

- **img**: 256 x 256으로 transformation된 사람의 이미지
- **img2bb_trans**: transformation matrix
- **bb2img_trans**: inverse transformation matrix



![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/ffb2e97e-11f5-4df7-9d48-610e29b96b3a)

coco joint x, y 좌표에 1로 채워진 열을 concat해서 coco_joint_img_xy1을 만든다. (transformation matrix를 곱해주기 위함)<br>
coco joint들을 transformation한 결과를 heatmap 사이즈로 scaling해서 coco_joint_img에 저장한다.<br>
transform_joint_to_other_db 함수를 사용해서 coco joint set을 SMPL joint set으로 변경한다. (SMPL에는 있고 coco에는 없는 joint의 경우 0으로 채워져 있음)
지금까지 가공된 coco_joint_img가 heatmap 범위 안에 있는지 여부를 coco_joint_trunc에 저장한다.

- **coco_joint_img**: transformation하고 heatmap 사이즈로 scaling한 SMPL joint list
    - list(num of SMPL joints x 3)
- **coco_joint_trunc**: coco_joint_img가 heatmap 범위 안에 있는지 여부
    - list(num of SMPL joints)

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/ace86ef0-447f-4192-b550-1b69a8c3bec1)

지금까지 가공한 이미지 **img**, joint **coco_joint_img**, 정상 여부 **coco_joint_trunc** 등을 input으로 모델에 넣는다. <br>
모델의 아웃풋으로 뽑힌 SMPL mesh를 이미지 상에 랜더링한다.


<br><br>
## 3DCrowdNet_RELEASE/main/model.py
demo 코드의 model은 get_model 함수를 통해 선언된다.

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/cd3d6705-a326-4b2a-9437-30bb695dd64d)
![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/aa25215f-c357-4422-b703-ca3c70c06120)

- backbone : crop된 이미지에서 early-stage image feature를 추출하는 네트워크
- Pose2Feat : early-stage image feature와 2D pose heatmap을 concat하여 crowded scene-robust image feature를 출력하는 네트워크
- PositionNet: crowded scene-robust image feature에서 $$P^{3D}$$를 예측하는 네트워크
- RotationNet: crowded scene-robust image feature와 $$P^{3D}$$를 가지고 파라미터 예측하는 네트워크

### Forward

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/d84cac79-8ec2-4fb3-9a62-56c1b870e578)

model의 forward에서는 논문에 나와있는 feature extractor와 joint-based regressor가 나와있다.

joint-based regressor에 사용되는 position_net을 조금 더 살펴보면 아래처럼 되어 있다.

#### PositionNet

우선 crowded scene-robust image feature $$F \prime$$에서 $$(J_c, D, 8, 8)$$차원의 feature map을 뽑습니다. ($$J_c$$: joint 개수, $$D$$: z 깊이) 다음으로 soft_argmax를 적용하여 joint coordinate $$(J_c, 8, 8, 8)$$를 뽑습니다.

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/ba1ca293-ae30-4cd2-8153-602429732342)

여기서 soft_argmax 함수를 뜯어보면 다음과 같다.
- soft_argmax : $$(J_c, D, 8, 8)$$차원의 feature map에서 3d heatmap 좌표를 뽑는 함수

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/39f436eb-8806-45df-92ef-6cb75911ee82)

최종적으로 PositionNet은 3d joint 좌표의 score(확률)를 구해서 함께 return합니다.

#### RotationNet

PositionNet에서 구한 3d joint 좌표와 score를 사용해서 GCN을 통과한 뒤 3d mesh estimation을 위한 파라미터들을 예측하는 네트워크입니다.