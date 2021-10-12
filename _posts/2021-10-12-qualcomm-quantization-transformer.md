---
title: "Understanding and Overcoming the Challenges of Efficient Transformer Quantization"
categories:
  - Paper Review
tags:
  - Uniform Quantization
  - BERT
  - Model Compression
---

## 
### 저자/학회 특이사항 : Qualcomm 
### Link : https://arxiv.org/abs/2109.12948v1

  - uniform quantization을 바로 Transformer에 적용하면, CNN과 달리 좋은 결과를 얻기 힘듭니다. 그것은 activation 때문인데, Weight 8bit Activation 32bit에서 깨지지 않던 accuracy가 weight 8/32bit, activation 8bit으로 가면 상당히 나빠지는 것을 논문에서 보여줍니다.
  - 여기서 한발 더 나아가서, Transformer 구조의 어떤 Activation이 많이 깨지는지를 확인했더니 FFN뒤에 activation이 residual connection에서 합쳐지는 부분이 8bit이 될때 가장 심각한 degradation을 보였습니다. exp 연산이 들어가는 softmax가 fixed-point와 어울리지 않기 때문에 이부분에서 degradation이 제일 심할것이라고 생각하고 있었는데, residual connection이 더 심했습니다. 아마도, layer가 많이 쌓여 있는 상태에서 Residual path로 흐르는 값이 상당히 크기 때문인것 같습니다.
  - layer별로, token별로, hidden vector의 dimension별로도 분석을 했더니, 결국 특정 layer에서 심하게 outlier가 발생하는 token과 vector의 특정 element가 존재한다는 사실도 알았습니다 (특히 [SEP]토큰). uniform quantization에서 outlier는 fixed-point 구간이 넓어지면서 전반적으로 상당한 손실로 이어질수 있습니다.
  - 이를 토대로 이 페이퍼는 mixed-precision으로 문제되는 구간의 activation을 보완하는 방법을 쓰거나, per-tensor대신에 per-embedding-group이라는 activation quant. 방법을 heuristic하게 제안하여 int8 압축상황에서도 상당히 높은 성능을 보여줄수 있도록 합니다. 
  - BERT-base에 대해서 아주 딥하게 분석을 하고 int8 적용시에 문제되는 부분을 해결한다는 점에서 해결 과정을 한번 관심있게 들여다볼 필요가 있다고 생각합니다. 한편으로는, 그만큼 int8 방법을 적용하는게 단순하지 않다는 뜻이 될 수 있을것 같고, NLG같은 모델을 하게 되면 또 완전 다른 분석 결과가 얻어질수 있다고 생각합니다.
