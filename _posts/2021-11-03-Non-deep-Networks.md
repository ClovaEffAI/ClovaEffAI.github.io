---
title: "None-deep Networks"
author: "Se Jung Kwon"
sidebar: true
author_profile: true
categories:
  - Paper Review
tags: // 
  - Image Classification
  - Object Detection
  - Efficient Architecture
  - CNN
---

### Link : https://arxiv.org/abs/2110.07641

 - 저자/학회 특이사항
   - Princeton, Intel
   - 2저자 Alexey는 YoLo v4의 저자, Jia Deng은 ImageNet paper의 1저자
  
#### Introduction
 - Deep Learning의 시대는 Layer를 얼마나 deep하게 쌓으면서도 Training을 잘되게 할 수 있느냐가 핵심인데, residual connection이 나온 뒤로는 수십개의 layer를 쌓는것이 당연해지고, Transformer 계열로도 이어지고 있다. 하지만 layer 수가 늘어난다는 것은 Dense한 연산을 병렬적으로 돌릴수록 좋은 Compuation 입장에서 좋지 않을 수 있는데, sequantial하게 이어져 있는 layer들을 위해서 계속해서 weight을 바꾸고, activation/normalization 등의 dense하지 않은 연산이 추가 되기 때문이다. 이것을 Training할때는 batch가 해결해주지만, inference할때는 batch를 만든 다는 것이 쉽지 않을 수도 있다. 
 - 본 논문은 이런 관점에서 depth에 대해 접근하고 있는 것 같다. 뒤에 실험 결과들을 보면 depth가 얉은 대신에, parameter 수는 꽤 많다. 즉, 하나의 layer가 뚱뚱하다는 뜻인데, width냐 depth냐 하는 백만번은 들어본 질문을 조금 더 공격적으로 던지는 듯 하다. 제안하는 네트워크의 이름은 ParNet(Parallel Network)이다. 12개의 layer depth만으로 SOTA 수준의 Image Classification / Object detection accuracy를 달성한다고 말한다. Figure 1은 Depth에 따라 그림을 그렸는데, VGG부터 시작해서 쭉 깊어지고 있는 전반적인 방향에 비해, ParNet은 depth는 그대로 두고 width만 늘린것으로 보인다 (물론 단순하게 width를 늘린것은 아니고, multi-stream network에 해당한다. architecture 그림을 보자.)
   - *We show, for the first time, that a classification network with a depth of just 12 can achieve accuracy greater than 80% on ImageNet, 96% on CIFAR10, and 81% on CIFAR100. We also show that a detection network with a low-depth (12) backbone can achieve an AP of 48% on MS-COCO.*
   - <img src="/assets/images/2021-11-03-Non-deep-Networks/f1.png" width="50%" height="50%" title="Figure 1" alt="Figure1"/> 


#### Architecture
 - <img src="/assets/images/2021-11-03-Non-deep-Networks/f2.png" width="100%" height="100%" title="Figure 2" alt="Figure2"/> 
 - Figure 2를 보면 단순히 Depth를 늘린 것은 아니고, Feature map size에 따라 여러 루트로 나뉘는 것을 볼 수 있다. (a)에서 각 네모의 너비는 channel수에 해당한다. 즉, 원래 size의 이미지를 downsampling하면서 작은 feature map + 더 많은 channel로 변환해가면서, 각 stage에서 multi-stream을 뽑아내고, 각각 연산한 후에 연산 결과를 합친다. 
 - ParNet은 기본적으로 VGG-style block을 가져왔다고 한다, 그냥 가져오면 잘 안될테니, RepVGG paper에서 'Structural reparameterization' technique을 들고 왔다. 
   - RepVGG는 올초에 나온 논문으로 VGG-style의 arch로 빠른 latency를 보이는 network를 제안하는데, 위 그림의 (b)에 나오는 것처럼 1x1 / 3x3으로 나눠서 학습은 시키고, 마지막에 fusion (reparameterzation)을 해서 합친다. 즉, 학습할때 1x1의 장점도 살리면서, dense한 연산이 더 가속이 잘되는 특징을 활용하기 위해 inference 전에 합쳐 버린다.
   - 하지만 3x3 conv 만으로는 shallow network에서 충분히 feature들을 잡아내지 못하는 것 같아서, Squeeze-Excitation(SE) block을 응용한 Skip-SE(SSE) block을 추가로 넣었다. 원래 SE block은 더 복잡한데, depth를 맞추기 위해서 간단하게 바꿔서 global avg pool - FC(1x1 conv) - sigmoid해서 채널별 가중치를 구해서 다시 원래 feature에 곱해주는 형태이다. ParNet의 block들은 기본적으로 input/output의 size가 같다.
 - Downsampling : resolution을 줄이고, channel을 늘린다. Appendix의 Figure A1 참고. 이 것도 multi-stream으로 이루어진 복잡한 conv. 연산이다.
 - Fusion : concat하는 형태로 정보를 합친다. Appendix의 Figure A1 참고.
 - SiLU : ReLU대신에 썼다. Sigmoid Linear Unit (SiLU), swish function을 말한다.
 - ParNet S/M/L/XL이 존재하며, 모두 Depth는 12이다. width, resolution, stream의 개수로 scaling을 해봤지만 stream의 개수는 결국 3으로 고정하고 resolution을 조금 올렸다. 어떤 scaling 법칙은 없는 것 같다. 약간 되는대로 만든 느낌이 든다. CIFAR10과 ImageNet은 일반적인 input size가 다르기 때문에 모델 구조도 살짝 다릅니다.

#### 결과
 - <img src="/assets/images/2021-11-03-Non-deep-Networks/result.png" width="100%" height="100%" title="Table 1-3" alt="Results"/> 
 - 우선 비교 대상은 ResNet이다. 그 이후에 상당히 다양한 연구들이 있었고, 이 network도 상당히 많이 바꿨음에도 ResNet하고만 비교하는 것은 fair하지 않은 것 같다. 결국 accuracy는 크게 나은 것 같지 않고, FLOPs, 파라미터수가 훨씬 많음에도 불구하고 speed가 빠르다 정도가 장점이 될 수 있을것 같은데 이것 또한 크게 차이가 나지 않는다 (Table 2). RTX 3090에서 쟨 숫자인데, 다양한 환경에서 재봐야 평가가 가능할 것 같다. 게다가 multi-stream 모델의 특성상, stream별로 모델 parallel로 실행 가능한데, 그렇게 실행을 해야지 Table 3에 있듯이 제대로된 속도가 나온다. 
 - 그 외, YOLOv4와 비교도 있고, 좀더 잘나온다 정도이다.
 - 다양한 실험 결과들이 있지만 크게 의미는 없는 것 같아서 여기서 생략. 특히 다양한 실험 결과가 CIFAR10에 몰려 있어서 제대로된 비교라기 어려울것 같다.
 - multi-stream이라는 구조를 가지고 depth대신 width라고 말하는게 살짝 의문이 들기는 한다. 결론부분에서 multi-processing chip을 말하는 것으로 봐서는 어느정도 인텔 CPU를 염두에 둔것 같기도 한데, 그러면 INT8기반의 AMX얘기를 좀 해야하지 않을까 싶기도 하다.
 
