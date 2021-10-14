---
title: "MobileViT: Light-weight, General-purpose, and Mobile-friendly Vision Transformer"
author: "Se Jung Kwon"
sidebar: true
author_profile: true
categories:
  - Paper Review
tags:
  - Vision Transformer
  - Light-weighted CNN
  - ImageNet
  - Apple
  - Mohammad Rastegari
  - Mobile ViT
  - On-device AI
---

Link : https://arxiv.org/abs/2110.02178?context=cs.LG

#### 저자/학회 특이사항
  - XNOR-net의 저자이고, xnor.ai을 만들어서 apple에 인수시킨, Rategari의 논문.
  - 퀄리티는 논문이긴 한데, Techinical Report로 낸게 흥미롭다. (ICLR 낼려다가 말았을까?)

#### Abstract & Introduction
  - Vision Transformer가 나온 뒤로 다양한 Variation들이 나왔고, ViT 그 자체를 더 키우려는 시도 (https://arxiv.org/abs/2106.04560), 효율적으로 만들려는 시도 (https://arxiv.org/abs/2012.12877), attention을 MLP로 대체하려는 시도 (https://arxiv.org/abs/2105.03404,  https://arxiv.org/abs/2105.01601,  https://arxiv.org/abs/2105.08050) 등이 있었고, 한편으로는 Convolution과 Transformer attention의 장점을 섞어보려는 시도들이 있었다 (https://arxiv.org/abs/2103.16302, https://arxiv.org/abs/2105.01883, https://arxiv.org/pdf/2106.04803v1.pdf). 
  - 이 논문은 CNN과 ViT의 장점을 섞고, mobile에 최적화 시키려는 시도를 한것으로 보인다. 우선, JFT같은 데이터셋은 구글이 아니니 쓰지 않았고, 어떻게 light-weight한 구조를 만들었는지, mobile에 대해 어떤 contribution이 있나 주로 봐야할것 같다. 논문에서는 CNN의 장점은 *spatial inductive biases and less sensitivity to data augmentation*라고 하고, Transformer의 장점은 *input-adaptive weighting and global processing*이라고 정리했고, 이것을 combine한다고 한다. 
  - 관심이 있는 영역이라 그대로 복사해 왔는데, FLOPs같은 숫자에 속지말라는 말. PiT (Pooling-based ViT)가 3배 적은 FLOP를 보이지만, 실제로 돌려보면 DeiT와 비슷한 속도를 보인다는 것. Dense한 연산이 연달아 일어나는 DeiT 대비, PiT에는 depthwise 1x1 conv에 기초한 풀링같은게 들어가서 그런가보다. 아니면 큰 레이어가 먹는 연산량과 작은 레이어가 먹는 연산량이 리니어한 관계가 아니라서 그럴까?
    - In Page2 : *Note that floating-point operations (FLOPs) are not sufficient for low latency on mobile devices because FLOPs ignore several important inferencerelated factors such as memory access, degree of parallelism, and platform characteristics (Ma et al., 2018). For example, the ViT-based method of Heo et al. (2021), PiT, has 3× fewer FLOPs than DeIT (Touvron et al., 2021a) but has a similar inference speed on a mobile device (DeIT vs. PiT on iPhone-12: 10.99 ms vs. 10.56 ms)*.
  - 하여간 핵심은 light-weight, general-purpose, low latency라고 한다. 하나씩 살펴보자.

#### light-weight architecture
  - <img src="/assets/images/2021-10-14-mobilevit/figure1b.png" width="100%" height="100%" title="Figure 1(b)" alt="Figure 1(b)"/> 
  - 그림을 보면 느낌이 오지만, Transformer보다는 CNN에 가깝다. MV2가 MobileNetV2의 block이라고 한다. 뒤애서 나오지만 MV2 block이 iPhone에서 가속이 잘된다고 한다. depth-wise conv.때문에 느려지지는 않는것 같다.
  - 3x3 conv.와 MobileNet v2로 spatial information이 농축된 feature map X를 만들고 나면, MobileViT block이 드디어 Patch로 잘라서 (그림에 있듯이, patch size(wh) * d channel) Transformer에 넣어준다. 즉, 이미 local spatial information을 응축시켜놓은 pixel들끼리 attention을 시키기 때문에 spatial + global information을 섞을수 있다고 말한다. (patch size는 상당히 작다. h=w=2니까 결국 4pixel씩 짜르고, d channel을 포함해서 하나의 hidden vector를 만드는듯.) 그리고 이미 locality를 반영한 정보를 만들고 feature map의 size를 줄인 다음에 Transformer를 돌리기 때문에 light-weight하다고 주장한다. 그리고 Transformer 결과로 나온 Feature map을 다시 1x1 conv를 거쳐서 원래의 channel 수로 복원시켜주고, 원래의 입력 feature map X와 concat해서 다시 한번 conv를 거쳐 결과를 내놓는 것이 MobileViT 블락 하나가 하는 일이다. 
  - <img src="/assets/images/2021-10-14-mobilevit/figure7r.png" width="100%" height="100%" title="Figure 7 right" alt="Figure 7 right"/> 
  - 논문에 있는 ViT계열과의 비교, ViT 계열이 Regularization이 중요하기 때문에 Advanced/Basic으로 구분해 두었다. MobileViT는 Basic인 부분이 흥미롭지만, 이미 ViT가 아닌것 아닐까?

#### general-purpose
  - 논문에서 general-purpose의 근거로 주장하는 것은 object-detection/segmentation 결과이다.
  - Table 1/2에서 MobileNet backbone 계열보다 더 좋은 결과를 보이는데, CNN에 global attention이 잘 섞여서 feature를 더 잘 뽑는게 아닌가 생각이 든다.

#### low-latency
  - 기본적으로 실험 환경은 iPhone 12 + CoreML.
  - <img src="/assets/images/2021-10-14-mobilevit/figure8.png" width="100%" height="100%" title="Figure 8" alt="Figure 8"/> 
  - 30FPS는 나와야한다고 할때, inference time이 33ms 안으로 들어와야 한다고 할때, 거의 모든 모델이 충분히 실행되는 것을 볼 수 있다. patch size가 커지면 complexity가 올라가면서 느려지기 때문에 patch size를 작게 가져가야한다고 한다. 근데 이것만으로 mobile-freindly라고 부를수 있나 모르겠다. 만약 논문이었다면 좀다 다양한 configuration의 실험을 요구할듯 하다.
  - 재미있는것 중 하나는 Table 3인데, MobileNet V2의 성능이 엄청나다. 이미 MobileNet을 위한 CUDA dedicated kernel이 잘 구축되어 있기 때문이라고 한다. 아마 모바일(bionic chip?) 안에 별도의 가속기가 있는듯 하고, Transformer를 위해서는 최적화가 덜 되어 있는 듯하다. DeiT/PiT보다는 빠른 성능을 보여줬다고는 하는데, 이것도 분석이 더 필요할듯 하다. 그리고 일부분 fusion을 통해 가속된 kernel도 존재한다(figure 1b에서 fusion이라고 쓰여 있는 부분). 
  - <img src="/assets/images/2021-10-14-mobilevit/table3.png" width="100%" height="100%" title="Table 3" alt="Table 3"/> 

#### 결론
  - 읽을 수록 왜 정식 paper가 아니고, technical report에 그쳤는지 알것 같다. 복잡한 문제가 꼬아놓았기 때문에 1:1 비교가 어렵고, contribution을 명확하게 뽑아내기 어렵다.
  - 이름은 MobileViT라고 했지만, CNN+attention 연구에 가깝다고 봐야할것 같고, 여러가지로 추가 질문이 생기는 실험 결과가 많다. 특히, ViT 계열 말고 CNN+Attention 연구들과 비교가 더 필요할것 같은데, 그런 쪽의 비교나 언급이 거의 없다. Apple의 AFT라던가, CoAtNet이라던가.
  - 그리고 뜬금없이 multi-scaler라는 것이 등장하는데, training할때 image size에 따라 성능이 달라지는 것은 잘 알려진 사실인데, cropped 되는 사이즈를 여러개를 두고 효율적으로 training에 사용했다고 한다 (3.2장). 요새 image classification 논문을 쉽게 비교하기 어려운 이유중에 하나에 이런 technique이 한 몫 하는 것 같다.

