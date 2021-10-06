---
emoji: 🪄
title: "[머신러닝] 딥러닝 영화 개인화 추천 - Part.2" 
date: '2020-08-22 23:00:00'
author: 코다
tags: 머신러닝
categories: 머신러닝
---

이어서 딥러닝 영화 개인화 추천 모델을 구현하면서 구축한 딥러닝 협업 필터링 모델 부분에 대한 코드를 살펴보면서 딥러닝 전체적인 흐름에 대해서 짚어보자. 앞에서 언급했듯이 코드는 다음 [링크](https://jyoondev.tistory.com/65?category=823946)에서 참고하여 모델과 전체적인 데이터 처리를 진행했고, 이후에 학습된 output에 대한 데이터 및 결과 후처리는 추가 구현했다. 해당 부분은 다음 파트에 다루도록 하겠다. <br>

## 모델 전체 코드 

```python
class NNCollabFiltering(nn.Module):
  def __init__(self, num_users, num_items, emb_size=100, n_hidden=10):
    super(NNCollabFiltering, self).__init__()
    self.user_emb = nn.Embedding(num_users, emb_size)
    self.item_emb = nn.Embedding(num_items, emb_size)
    self.lin1 = nn.Linear(emb_size*2, n_hidden)
    self.lin2 = nn.Linear(n_hidden, 1)
    self.drop1 = nn.Dropout(0.1)
    
  def forward(self, u, v):
    U = self.user_emb(u)
    V = self.item_emb(v)
    x = F.relu(torch.cat([U, V], dim=1))
    x = self.drop1(x)
    x = F.relu(self.lin1(x))
    x = self.lin2(x)
    return x
```

<br>

## Step1. 임베딩 - nn.Embedding() 

위 코드를 보면 먼저 User와 Item에 관한 임베딩으로 시작한다. 파이토치에서 임베딩 벡터를 사용하는 방법은 크게 두가지가 있는데 그 중 위의 `nn.Embedding()` 은 embedding layer를 만들어서 훈련 데이터로부터 처음부터 임베딩 벡터를 학습하는 방법이다. <br>

먼저 임베딩 층의 입력으로 사용하기 위해서는 정수 인코딩이 되어 있어야 한다. 즉 다음과 같은 단계를 따른다. <br>

> 어떤 단어 -> 고유한 정수로 인코딩 -> 임베딩 층을 통과 -> 밀집 벡터 

즉, 고유한 정수 인코딩 값에 대해서 밀집 벡터(dense vector)를 맵핑해주는 것이다. 이 밀집 베터가 흔히 알고 있는 임베딩 벡터이다. 임베딩을 시킨다는 것은 어떠한 단어에 대한 고유 인코딩 정수값을 인덱스로 가지고 있는 룩업 테이블에서 해당 임베딩 벡터 값을 가져오는 것이다. 또한 <mark>이 테이블은 단어 집합만큼의 행을 가지고 있으므로 모든 단어들은 고유한 임베딩 벡터를 보유하게 된다.</mark> 이러한 벡터 값을 담고 있는 룩업 테이블을 생성하는 것이 `nn.Embedding()`의 역할이다. <br>

![image](https://user-images.githubusercontent.com/63405904/110482824-6bad5d80-812c-11eb-9da2-750e1946c80b.png)

위의 그림을 참고해보면 단어 'great'에 대한 임베딩 벡터가 4차원인 것을 확인할 수 있다. 해당 차원값은 parameter로 넘겨줄 수 있는 부분이다. 이렇게 생성된 임베딩 벡터는 모델의 입력이 되고, 역전파 과정을 거치면서 바로 이 임베딩 벡터값이 학습 되는 것이다. <br>

### > 코드에서 임베딩

위 코드에서 임베딩이 어떻게 이루어지고 있는지 살펴보자. 코드를 살펴보면 임베딩 관련한 부분에 다음과 같이 있다. <br>

```python
#def __init__ 메소드 내: 
  self.user_emb = nn.Embedding(num_users, emb_size)
  self.item_emb = nn.Embedding(num_items, emb_size)
```

`nn.Embedding()` 에 넘겨지는 parameter는 크게 **2가지**가 있다. 1) 테이블 사이즈 (단어 및 데이터 갯수) 2) 임베딩 사이즈 (embedding vector  차원). <br>

영화에서 모델에 넣어서 학습할 데이터는 사용자 user와 영화 item이다. 이 두개에 대한 임베딩 테이블을 생성하기 위해서 인코딩 하며 중복없이 뽑아낸 user 와 item 리스트의 크기와 임베딩 사이즈를 결정해서 `nn.Embbeding()`을 호출한다. 그럼 임베딩 테이블이 생성되어 각각 user_emb 와 item_emb에 저장된다. <br>


## Step2. Linear Layer 생성 - nn.Linear()

다음 코드에서는 Linear layer를 생성한다. 딥러닝의 핵심인 신경망(neural network) 층을 쌓아올려서 학습을 진행한다. 그때 필요한 신경망 층을 생성하는 역할을 한다. 딥러닝을 위한 신경망은 기본적으로 선형회귀분석을 기본으로 하기 때문에 선형변형 함수로 층을 쌓는다. <br>

![image](https://user-images.githubusercontent.com/63405904/110482886-7d8f0080-812c-11eb-9b12-6c65a734be69.png)

파이토치에서 제공하는 [document](https://pytorch.org/docs/stable/nn.html#linear-layers)를 살펴보면 위와 같은 선형변형 함수를 사용하는 것을 확인할 수 있다. 선형결합은 보존하는 선형변형 함수를 생성하고 원하는 **in_feature**와 **out_feature**의 사이즈를 parameter로 넘긴다. <br>

### > 코드에서 layer 생성

```python
#def __init__ 메소드 내:
	self.lin1 = nn.Linear(emb_size*2, n_hidden)
  self.lin2 = nn.Linear(n_hidden, 1)
```

위 코드는 입력 차원이 emb_size의 두배인 input sample에 대해서 n_hidden 사이즈 만큼의 차원으로 선형변형을 하는 linear layer 하나와, n_hidden 사이즈의 input sample에 대해서 1로 선형변형을 하는 linear layer 두개를 생성한다. <br>

## Step3. 모델 일반화 - nn.Dropout()

Dropout은 모델을 일반화 기법으로 일부 파라미터를 학습에 반영하지 않는 것이다. Validation과 test 시에는 적용하지 않고 train 시에 dropout을 적용하는데, 일종의 정규화 기법이라고 볼 수 있다. 모델을 학습할 때 과적합(overfitting)의 위험을 줄이고, 학습속도를 개선하는 문제를 해결하기 위한 방법이다. 모델을 학습할 때 지나치게 학습 데이터에 대한 높은 정확도를 보이기 보다, 범용적으로 사용될 수 있도록 overfitting 문제를 피하기 위해서 고안된 해결책 중 하나이다. 일반적으로 신경망의 층이 깊어지고, 학습률이 작을수록 overfitting이 될 가능성이 높다. <br>

이중 본 코드에서 사용하고 있는 모델 일반화의 방법은 드롭아웃 Dropout이다. 신경망 모델이 지나치게 복잡해질 때, 뉴런의 연결을 임의로 삭제하여 전달하지 않도록 떨어뜨리는 역할을 한다. 다만, 테스트를 할 때에는 모든 뉴런을 사용하기 때문에 반드시 학습시에만 드롭아웃을 적용해야 한다. <br>

![image](https://user-images.githubusercontent.com/63405904/110482706-47ea1780-812c-11eb-9908-5686512aae8c.png)

파이토치 [document](https://pytorch.org/docs/stable/generated/torch.nn.Dropout.html#torch.nn.Dropout)를 보면 `nn.torch` 모듈에서 드롭아웃 또한 지원을 한다. <br>

![image](https://user-images.githubusercontent.com/63405904/110482789-5fc19b80-812c-11eb-960c-8f7da3b2597f.png)

파이토치에 제공하는 도큐멘트를 살펴보면 학습시 무작위로 몇개의 뉴런들에 대해서 *p* 확률만큼  'zeros' 시킨다고 나와있다. 이때 *p*는 parameter로 주어지는 확률 변수이고, default는 0.5이다. `forward`함수가 호출될 때마다 적용되도록 되어 있다. <br>

### > 코드에서 Dropout

```python
#def __init__ 메소드 내:
	self.drop1 = nn.Dropout(0.1)
```

본 코드를 살펴보면 nn.torch 모둘에서 Dropout 함수를 호출하고 확률 변수를 0.1로 주었다. <br>

## Step4. 활성화함수 - torch.nn.functional.relu()

다음 딥러닝 신경망 모델 구축에서 중요한 부분은 활성화 함수(Activation Function)이다. 활성화 함수는 최종출력 신호 후, 다음 뉴런으로 보낼지 말지를 결정하는 함수이다. 즉, 특정 뉴런이 다음 뉴런으로 신호를 보낼 때 입력신호의 어떠한 기준에 따라서 보내고 보내지 않는지를 결정하도록 하는 것이다. 딥러닝에서는 뉴런들을 다음 레이어로 전달할 때 비선형 함수를 통화시킨 후 전달하도록 하는데 이때 사용되는 함수가 활성화 함수이다. <br>

딥러닝 학습의 핵심은 이름에서 볼 수 있듯이 깊게 층을 쌓아서 그 층을 통과하면서 학습되는 것인데, 선형함수를 사용하게 되면 층을 깊게 하는 의미가 줄어들게 된다. 해당 설명은 [밑바닥부터 시작하는 딥러닝] 책의 한 부분을 인용하도록 하겠다. <br>

> 선형합수인 h(x)=cx를 활성화함수로 사용한 3층 네트워크를 떠올려 보세요. 이를 식으로 나타내면 y(x) = h(h(h(x)))가 됩니다. 이는 실은 y(x)=ax와 똑같은 식 입니다. a=c3이라고 하면 끝이죠. 즉, 은닉층이 없는 네트워크로 표현할 수 있습니다. 뉴럴네트워크에서 층을 쌓는 혜택을 얻고 싶다면 활성화 함수로는 반드시 비선형 함수를 사용해야 한다. 
>
> -밑바닥부터 시작하는 딥러닝-

이러한 역할을 하는 활성화 함수는 많은 종류가 있다. **1) 시그모이드 함수 2) tanh 함수 3) ReLU 함수.**  본 코드에서는 가장 많이 사용되는 활성화 함수인 ReLU 함수를 사용했다. <br>

![image](https://user-images.githubusercontent.com/63405904/110482937-8b448600-812c-11eb-9672-04dc7c7e5928.png)

ReLU 함수를 살펴보면 $x > 0$ 이면 기울기가 1인 직선이고 $x < 0$이면 함수값이 0이 된다. 따라서 다른 활성화함수에 비해서 굉장히 간단하고 빠르다. 해당 함수는 양수에서는 Linear function 과 같은 모습을 보이지만 음수의 경우 0으로 버려지므로 non-linear 한 함수로 작동하여 layer를 깊게 쌓을 수 있는 장점을 가진다. <br>

### > 코드에서 활성화함수

```python
#def forward 내
	x = F.relu(torch.cat([U, V], dim=1))
```

선형함수가 linear layer에 들어가기 전에 비선형 함수를 거친다. 위의 코드에서 F는 `torch.nn.funtional`모듈이며 모듈 내에 있는 `relu()`를 사용하고 있다. <br>

<br>
<br>

**<small> [참고 자료]: https://wikidocs.net/64779, https://tutorials.pytorch.kr/beginner/blitz/neural_networks_tutorial.html, https://pytorch.org/docs/, https://yeomko.tistory.com/39, https://reniew.github.io/12/, https://eda-ai-lab.tistory.com/405, https://jyoondev.tistory.com/65?category=823946, https://sacko.tistory.com/45</small>**

```toc
```



