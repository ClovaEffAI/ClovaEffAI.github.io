---
title: "Block Pruning For Faster Transformers"
categories:
  - Paper Review
tags:
  - Structured Pruning
  - Movement Pruning
  - BERT
  - Model Compression
---

## 
### 저자/학회 특이사항 : HuggingFace, EMNLP 2021
### Link : https://arxiv.org/abs/2109.04838

 - Movement-based Pruning은 Pruning의 기준을 static한 weight의 magnitude로 보는 것이 아니라, Training단계에서 gradient를 보면서 0에서부터 멀어지려는 힘이 얼마나 쎈지를 보는 방법입니다. 여기서는 fine-tuning단계에서 20epoch 정도 task-specific data를 가지고 pruning을 적용했습니다.
 - 기존 movement-pruning 연구에서 주로 fine-grained pruning에 적용되었던 것을, 이 논문에서는 32x32 block(attention), 1 dimension(FFN) 단위로 coarse-grained로 적용했습니다. 즉, attention은 block별로 mask가 생기고, FFN은 마치 cnn channel pruning처럼 한 줄이 날라가는 형태가 됩니다. 때문에 특별한 가속기 없이도 충분히 가속이 가능하며, 실제로 2-3x 빨라지는 결과를 얻었습니다.
 - 사실 block-based 혹은 structured pruning을 별로 좋아하지 않습니다. 엄청난 가속 성능을 얻기에는 sparsity가 높지 않다는 점이 단점이라고 생각했었는데, movement-pruning + BERT fine-tuning 이 꽤나 괜찮은 결과가 보였다는 것이 흥미롭고, 기존의 컴퓨팅 시스템에서 잘 돌아가는 기술이기 때문에 상당히 주목할만하다고 판단하고 있습니다.
 - 한편으로는, BERT가 워낙 Sparse한 측면이 있어서, 그런 결과를 얻었다고 생각할수도 있습니다. 개인적인 경험으로도 BERT는 task에 따라서, 상당수 parameter가 importance가 떨어지는 경우가 많았습니다.
 - BERT에 대해서만 했기 때문에 다른 variation들, 혹은 NLG 모델에 대해서는 다시 생각해볼 필요가 있지만, 2가지 측면에서 레슨을 얻을수 있는데. 
   1. 열심히 Knowledge Distillation을 돌린 TinyBERT같은 variation보다 나은 결과를 Pruning만으로 얻을수 있음. 
   2. BERT-base를 압축하는 것보다 BERT-large 모델을 더 많이 압축하는 것이 더 효율적일수 있음. 
