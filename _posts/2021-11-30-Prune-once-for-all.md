---
title: "Prune Once for All: Sparse Pre-Trained Language Models"
author: "Se Jung Kwon"
sidebar: true
author_profile: true
categories:
- Paper Review
tags:
- Language Model
- Pruning
- Knowledge Distillation
- Transfer Learning
- Model Compression
- Quantization
- Once for all
---

### Link : https://arxiv.org/pdf/2111.05754.pdf
 - 저자/학회 특이사항
   - Intel Labs, Israel
   - ENLSP NeurIPS Workshop 2021
 - 제안하는 방법에 대해서는 의구심이 조금 생기지만, 2/3/4장만 잘 읽어도 Pruning이나 Knowledge distillation에 대해서 공부하기는 좋을듯.
  
#### Introduction
 - **Prune One for All** (Prune OFA)라는 방법을 제안한다. 기본적으로 BERT처럼 Pre-training 후에 작은 downstream dataset에 대해서 transfer fine-tuning하는 모델들을 대상으로 한다. 아마도 Generative Model (e.g. GPT)에 대해서는 적용하기 힘들것이라고 생각하는데, 아직까지 GPT를 Sparse하게 만들었다는 말은 들어보지 못했다. BERT는 정보를 모으는 Encoder다보니 비교적 쉽게 압축이 되는 편이다. 
 - main contribution은 크게 세가지다.
   1. sparse pre-trained model을 만드는 architecture-agnostic method를 제안한다.
   2. downstream task로 sparse pre-trained model을 sparse하고 quantized된 형태로 잘 fine-tuning하는 효과적인 방법을 제시한다.
   3. compression library를 publish한다. 
 - 첫번째 contribution은 좀 거창하다고 생각되는게, BERT/DistilBERT만 가지고 architecture-agnostic하다고 주장하기에는 무리가 있다. 기껏해야 Roberta정도? 그나마 Roberta도 경험상, BERT와 경향이 많이 달랐었다기 때문에 적용이 될지 잘 모르겠다. KD나 Pruning 방법의 우수성이, 모델의 redundancy나 training 난이도와 함께 엮여서 과대 포장되는 경우가 많은데, 이 논문도 그렇게 읽힌다.
 - (Gordon et al., 2020)이라는 Paper가 자주 언급되는데, *Compressing BERT: Studying the Effects of Weight Pruning on Transfer Learning*라는 제목의 paper로, pre-training 단계에서도 pruning해보고, fine-tuning 이후에도 해보면서 여러 Insight를 얻어보고자 하는 논문이다 (https://arxiv.org/pdf/2002.08307.pdf). Gordon의 논문의 여러 Insight 중에서, pre-training 단계에서 pruning을 한것이, downstream fine-tuning 단계로 가져갈수 있다는 점을 본 논문이 주목한것 같다. 또한 (Chen at el, 2020)에서도 BERT에서의 Lottery ticket을 찾아보려고 하면서, pre-trained model의 lottery를 fine-tuning 단계로 들고 갈수도 있다는 점을 분석했는데, 이 부분도 중요한 insight로 다루고 있다.

#### Prune Once for All
 - Pruning 방법은 (Zhu and Gupta, 2018)의 Gradual Mangnitude Pruning (GMP) 방법과 (Renda et al., 2020)의 Rewinding 방법(Learning Rate Rewinding, LRR)을 참고하는 듯 하다. unstructured (fine-grained) 방법을 사용했으나, 가속관점의 이슈는 전혀 다루지 않았다.
 - KD 방법은 우리가 아는 기본적인 방법을 그대로 사용했지만, Sparse Pre-trained model을 만들면서 한번 Teacher 모델이 사용되고, Fine-tuned sparse model for downstream task를 만들기 위해서 한번 더 들어간다 (모든 task별로 fine-tuned BERT teacher 모델이 다 필요합니다!). 다만 Pre-trained model은 loss가 MLM을 사용하기 때문에 'Teacher preparation'단계에서 loss를 (아마도) CLS 토큰에 맞게 fine-tuning하는 과정이 한번 더 들어가는 것 같다.
 - <img src="/assets/images/2021-11-30-Prune-once-for-all/f1.png" width="100%" height="100%" title="Figure 1" alt="Figure1"/> 
 - 위의 그림에서 두번째 'Fine-tuned pre-trained LM'은 KD를 위해서 pre-trained model의 loss를 바꾼 것을 뜻한다. 뒤의 fine-tunign과 헷갈리면 안된다.
 - Sparse pre-trained LM의 sparsity pattern은 fine-tuning 단계에서 유지된다. 


#### Experimental Setup
 - <img src="/assets/images/2021-11-30-Prune-once-for-all/t1.png" width="100%" height="100%" title="Table 1/2" alt="Table 1/2"/> 
 - Table 1에서 Transfer with KD가 없는 Chen/Gordon의 연구와 비교했을때 비슷하거나 더 좋은 결과를 얻을 것을 볼 수 있는데, 더 높은 sparsity를 보인다. transfer 단계에서 KD를 뺐기 때문에 sparse model을 만들면서 KD를 사용하는 것이 의미가 있는 것으로 보인다.
 - 흥미로운 점은 QAT까지 적용한 모델이 있는데, 추가적으로 한번더 KD가 포함된 QAT step을 돌리고 Q8BERT와 비슷한 방법을 사용했으며, activation을 위해서는 asymmetric을 사용했고, embedding 압축은 안했다고 한다. 
 -  <img src="/assets/images/2021-11-30-Prune-once-for-all/t3.png" width="100%" height="100%" title="Table 3" alt="Table 3"/> 
 -  Table 3에서는 그냥 downstream task 단계에서 KD를 포함해서 pruning한 것과 비교했다. 조금 더 좋은 결과를 보이는 것을 볼 수 있다.

#### 총평
 - architecture-agnostic이라는 말이 어떻게 review를 통과했을까 생각해보니, 아차 workshop 논문이었다.
 - Once-for-all 이라는 말을 쓸 수 있는지 잘 모르겠다. 결국 distillation을 위해서 이미 만들어진 fine-tuned 모델을 사용했는데..?
 - 큰 모델부터 KD를 쏟아부으면 fine-tuning 단계에서만 하는것보다 더 좋은 결과를 얻을수 있다는 뜻인데, 어찌보면 맞는 말이라 novelty가 있나 싶다.
