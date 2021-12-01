---
title: "Sparse is Enough in Scaling Transformers"
author: "Se Jung Kwon"
sidebar: true
author_profile: true
categories:
- Paper Review
tags:
- Language Model
- Efficient Transformer
---

### Link : https://arxiv.org/pdf/2111.12763.pdf
 - 저자/학회 특이사항
   - Google Research in Europe, Reformer 저자도 있고 (Łukasz Kaiser), Performer 저자도 있고(Afroz Mohiuddin), 계속 Efficient Transformer를 연구하던 조직인듯.
   - NeurIPS 2021 Accepted Paper
 

#### Introduction
  - Model Compression을 많이 공부한 입장에서 'Sparse'라는 단어는 종종 Pruning으로만 생각하기 쉬운데, 여기서 이 연구는 Efficient Transformer 연구라고 봐야할 것 같고, 여기서 sparse는 모델은 크지만 일부만 활성화된다는 관점의 sparse, 즉, Mixture of Experts와 비슷한 관점이다. 다만 여러개의 weight중에 하나를 부르는 것이 아니라, weight의 일부만 활성화 시키는 것이다. 특히 실제적인 decoding 속도를 계산한 것을 두고 공부해볼 필요가 있다고 생각했다.
  - 논문의 시작은 Transformer 연산에 상당한 시간과 비용이 들고 있다는 점을 지적하면서 시작한다. 그래서 *Scaling Transformer*라는 것을 제안하는데, MHA에 대한 sparse한 구조, 그리고 FF에 대한 Sparse한 구조를 적용하여 efficient한 transformer를 제안한다고 한다. *Scaling*이 붙은 이유는, hidden size에 제곱(2승)으로 늘어나는 complexity를 1.5승으로 줄였기 때문에 더 scaling 하기 쉬워서 인것 같다. 더 나아가서 *Terraformer*라는 것도 만들었는데, 이것은 심지어 activation을 절약하기 위한 reversible layer까지 적용해서 memory efficiency를 높여서 scalability를 더 높였다.

#### Sparse FF
  - 가장 중요한 observation은 FF의 두개의 FC layer 사이에 있는 ReLU(!!)에 의해 엄청나게 많은 0이 만들어진다는 것이다. (왜 GeLU가 아니냐고 물을수 있지만, C4 dataset에 대해서 ReLU나 GeLU나 별차이 안난다고 얘기하고 레퍼런스 걸고 넘어간다.)
  - dynamic sparsity라고 하는 핵심 기술은 training할때는 weight을 다 training하지만, inference할때는 controller가 미리 activation을 보고 일부 weight만 불러오도록 하는 것이다 (아래 그림 참고). 어차피 ReLU에서 다 0이 될거라면 0이 되지 않을 놈들을 미리 controller가 골라내도록 하자는 말이다.
  - <img src="/assets/images/2021-12-01-Scaling Transformers/f2_.png" width="100%" height="100%" title="Figure 2" alt="Figure 2"/>
  - controller는 low-rank의 곱셈들로 이루어지는데, 큼직큼직하게 미리 연산을 해보고 결과를 가늠한다고 생각해볼수 있을것 같다. 그리고 그 결과를 가지고 straight-through-gumbel-softmax를 통해 training 할때는 0/1을 골라내고 (0/1이 discrete하기 때문에), inference할때는 argmax를 통해 N개의 활성화되는 activation을 골라낸다.
    - forward pass에서는 그냥 argmax로 가고, backward 탈때 softmax로 approximation한다. 이것을 straight-through-gumbel-softmax라고 하는듯.
  -  <img src="/assets/images/2021-12-01-Scaling Transformers/f3.png" width="100%" height="100%" title="Figure 3" alt="Figure 3"/>
  -  Table 2에서 'Sparse FF 64/128'은 N=64/N=128을 의미한다. 즉, hidden vector 1024개 에서 4096개로 뻥튀기 되었다가 다시 1024로 projection될때, 64/128개만 보겠다는 뜻이다. 그러면 low-rank는 4096/64=64, 4096/128=32가 된다. 그만큼 빨라지는 것을 확인할 수 있다. N이 늘어나는 만큼 low-rank가 줄어들어서인지 N=64/128의 시간은 비슷하다.
  -  그리고 그 결과, baseline 대비 N=64일때 더 좋은 결과인것을 볼 수 있다. 오히려 128일때 low-rank 값이 줄어든면서 성능이 나쁜것도 확인할 수 있다. 때로는 거추장스러운 연산을 치워버리는 것이 도움이 될 수 있는데, 그런 관점에서 납득이 가는 결과이다.


#### Sparse QKV layer
  - decoding speed 자체는 FF가 dominant하긴 하지만, FF를 줄이고 나면 당연히 그 다음으로 QKV/output 연산을 줄여야할 차례이다. 하지만 여기에는 ReLU가 없기 떄문에 위와 같은 적용은 불가능하다.
  - Sparse Attention 연산을 만들어내기 위해서, hidden vector를 *S*개로 쪼개서 나눠서 연산하도록 한다. 당연히 그러면 일부분의 hidden vector 안에서만 attention이 되기 때문에 문제가 될텐데, 이것을 막기 위해 multiplicative layer라는 것을 통해 hidden vector를 어떤 형태로 permutation을 해주고 conv layer를 통해 attention연산에 들어가는 Q/K/V를 만들도록 한다.
  -  <img src="/assets/images/2021-12-01-Scaling Transformers/f4.png" width="100%" height="100%" title="Figure 4" alt="Figure 4"/>
  -  Multiplicative dense layer는 주어진 hidden vector를 S * (hidden vector size / S)로 permutation한다. 어떤 부분을 봐도 전체 vector를 보도록 정보를 뭉쳐주는 작업으로 보인다. 보통 S=16이라니까, hidden vector 1024에 대해서 16x64 size의 중간 matrix를 만들어내는 것 같다. 그림에서 알수 있듯이, 이 matrix의 크기도 decomposition 된 형태로 작은 편이다.
  -  Convolution layer는 (batch, length, S, M)으로 이뤄진 activation에 대해 3x3 filter가 M개가 있다고 생각하고, (length x S) feature map에 대해서 convolution을 수행한다. 이때 결국 sequence length에 대해서 local하게 다른 vector의 정보를 보기 때문에 여기서 *the model can incorporate more context into this computation*라고 한다 (아직 attention을 수행하지 않았음.)
  -  그 결과 같은 size의 tensor가 튀어나오는데 이것을 M개의 head로 쪼개진 S개의 head라고 보고 multi-head attention에 넣는다.
  -  output layer는 날려버렸는데, decoding time에 도움이 안되는것 대비 성능 향상이 없다고 판단한것 같다.
  -  이렇게 해서 나온 결과가 Figure 5에 있는데, log-perplexity가 위와 같은 구조로 했을때 크게 차이나지 않는 것을 볼 수 있다. 특히 conv filter의 크기가 3x3일때가 1x1보다 좋은데, Q/K/V 만드는 단계에서부터 조금씨 다른 vector를 Local하게 보는 것에 따른 장점으로 보인다.
  -  loss를 구할때도 비슷한 구조로 S로 나눠서 했더니, 아주 약간만 하락했고 대신 성능은 더 빨라졌다고 한다.
  -  <img src="/assets/images/2021-12-01-Scaling Transformers/t6.png" width="100%" height="100%" title="Table 6" alt="Table 6"/>
  -  그렇게 해서 나온 최종 결과이다. T5-base로 17B 모델을 가져와서 37배 빠르게 동작하는 모델을 만들수 있었다고 한다. 
  -  Terraformer에 대한 설명은 조금 디테일이 적은 느낌이라 생략. Reformer와 비슷한 방법을 사용하여 Revisible logic을 이용하여 메모리 사용량을 줄이고, long sequence에 대응될수 있도록 했다.
