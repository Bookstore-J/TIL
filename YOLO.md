# YOLO

### **Object Detection (물체 검출)**

이미지 내 물체의 **위치**와 **종류**를 함께 찾는 기술

1) Image Classification 과 Localization을 구분 짓는 Two-stage Detecto

- 이미지를 여러번에 걸쳐서 확인하며 동작하기 때문에 실시간 물체 검출이 어렵다
- ex) Sliding Window, Region Proposal 방식

**2) 두 과정을 구분 짓지 않는 One-stage Detector → YOLO!**

### **YOLO 장점**

1. YOLO는 이미지에 대해 빠른 속도로 Object Detection을 수행할 수 있다.

→ localization과 classification을 **하나의 회귀 문제**로 보아서 그럼.

<aside>
💡 **Input image를 7 x 7 그리드로 나눈후, 하나의 그리드 셀에서 2개의 (BBox 좌표와 Confidence score) 그리고 Conditional class probabilities를 예측하므로, 최종 예측값의 dimension은 (7 x 7 x 30)입니다.
* 30 = B(2) * (x,y,w,h,Conf) + Conditional class probabilities(20)**

</aside>

(참고) Fast YOLO는 YOLO의 구조를 간단하게 하여 처리 속도가 3배 이상이 되도록 만든 모델입니다.

1. YOLO는 이미지 전체를 본다. 
    
    **클래스의 모양에 대한 정보와 주변 정보**까지 학습하여 처리한다.
    
    Fast R-CNN는 주변 정보까지는 처리하지 못한다. 그래서 아무 물체가 없는 배경에 반점이나 노이즈가 있으면 그것을 물체로 인식한다. 이를 **background error**라고 한다.
    
2. YOLO는 객체의 Generalizable Representation을 학습한다는 점입니다. 
    
    Generalizable Representation란? 일반화가 가능한 특징을 학습
    
    → 실제가 아닌 미술 작품에서도 높은 성능을 보임.
    

### **YOLO 방식**

이미지를 7X7 격자무늬의 그리드 셀로 나눈뒤 각 그리그 셀별로 2개의 BBox를 예측한다. 

결과적으로 1장의 이미지에 대해 98개 BBox(7*7*2와 조건부 클래스 확률를 예측하게 되고,

98개의 class specific confidence score에 대해 각 20개의 클래스를 기준으로 **non-maximum suppression**을 하여, Object에 대한 Class 및 bounding box Location를 결정한다.

1) Input image를 S*S 크기로 Grid를 나눕니다. 한 Grid Cell안에 어떤 객체의 중심이 포함되면, 해당 Grid Cell이 그 객체에 대해 책임을 지고 탐지를 수행합니다.

2) 각 Grid Cell*은 **B*** 개의 Bounding Box와 Confidence Score를 예측합니다. 
3) 각 Grid Cell은 ***C*** 개 클래스에 대한 Conditional Class Probabilities를 계산합니다. 각 Bounding Box 안에 있는 객체가 어떤 클래스일지에 대한 조건부 확률입니다.

### **YOLO 구조**

YOLO는 24개의 CNN layer를 가지고 있다.

Convolutional Layers 뒤에 있는 두 개의 Fully Connected Layers는 Output Probabilities와 Coordinates 예측합니다

### **Training phase**

ImageNet의 1000-class Competition Dataset으로 **첫 20개의 CNN Layers (=DarkNet)**를 Pretraining합니다. **Pretrained 된 네트워크에 CNN Layers와 FCN Layers를 추가했을 때 네트워크의 성능이 향상된다는 연구가 있었습니다**. 그래서 Pretrained CNN Layers 뒤에 4개의 CNN Layers와 2개의 FCN Layers를 추가합니다.이때 새롭게 추가된 6개의 Layers는 Randomly Initialized 가중치를 갖게 됩니다. 마지막으로 Input Resolution을 224 x 224에서 448 x 448로 증가하여 모델의 성능을 끌어올렸습니다.

YOLO는 마지막 Layer에서만 Linear Activation Function을 사용하고, 나머지 Layers에는 Leaky Rectified Linear Unit(ReLU) Activation Function를 사용합니다.

### **Loss Function**

![출처: [https://herbwood.tistory.com/13](https://herbwood.tistory.com/13)](YOLO%208f661e87732142728dc177a13f5c1ab8/Untitled.png)

출처: [https://herbwood.tistory.com/13](https://herbwood.tistory.com/13)

 YOLO v1 모델은 regression 시 주로 사용되는 **SSE(Sum of Squared Error)**를 사용합니다. 위의 그림에서 볼 수 있듯이 **Localization loss, Confidence loss, Classification loss**의 합으로 구성되어 있습니다. 

### **한계**

***1.*** 각 Grid Cell은 오직 B개의 Bounding Box를 예측하고 **하나의 클래스**만 갖을 수 있습니다. Spatial Constraints에 의해 가까이 있는 **객체의 수에 대한 예측 제한이 발생**합니다. YOLO는 주로 하나의 Cell에서 2개의 객체만 탐지하므로 모든 객체를 탐지하지 못하는 문제가 발생합니다.

***2.*** YOLO는 특정한 데이터로만 학습합니다. 따라서 BBox의 가로 세로의 비가 Unusual할 경우 탐지 성능 저하됩니다.

***3.*** Loss Function은 Bounding Box의 크기를 고려하지 않습니다. Bounding Box의 width와 height의 증가율을 감소시킨 것이 궁극적으로 작은 Bounding Box가 Error에 더 민감한 문제를 해결하지 못합니다.

### 참고

NMS(Non Maximum Suppression): Object detection의 Overlap problem을 해결하기 위한 방법

여러 후보 BBox에서 최적의 BBox를 추출하는 과정

1) 가장 높은 Objectiveness score를 가지는 BBox를 선택

2) overlap(IOU 기준) 되는 BBox 제거

### Ref)

[https://arxiv.org/abs/1506.02640](https://arxiv.org/abs/1506.02640)

[https://herbwood.tistory.com/13](https://herbwood.tistory.com/13)

[https://herbwood.tistory.com/14](https://herbwood.tistory.com/14)

[https://bkshin.tistory.com/entry/논문-리뷰-YOLOYou-Only-Look-Once](https://bkshin.tistory.com/entry/%EB%85%BC%EB%AC%B8-%EB%A6%AC%EB%B7%B0-YOLOYou-Only-Look-Once)

[https://mr-waguwagu.tistory.com/35](https://mr-waguwagu.tistory.com/35)
