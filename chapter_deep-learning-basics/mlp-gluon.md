# Concise Implementation of Multilayer Perceptron

# 멀티레이어 페셉트론의 간결한 구현

Now that we learned how multilayer perceptrons (MLPs) work in theory, let's implement them. We begin, as always, by importing modules.

Multilayer perceptron (MLP)가 어떻게 작동하는지 이론적으로 배웠으니, 직접 구현해보겠습니다. 우선 관련 패키지와 모듈을 import 합니다.

```{.python .input}
import sys
sys.path.insert(0, '..')

import d2l
from mxnet import gluon, init
from mxnet.gluon import loss as gloss, nn
```

## The Model

# 모델

The only difference from the softmax regression is the addition of a fully connected layer as a hidden layer.  It has 256 hidden units and uses ReLU as the activation function.

Softmax regression과 유일하게 다른 점은 hidden 레이어로 fully connected 레이어를 추가한다는 점입니다. 이 hidden 레이어는 256개의 hidden unit을 갖고, activation 함수로 ReLU를 사용합니다.

```{.python .input  n=5}
net = nn.Sequential()
net.add(nn.Dense(256, activation='relu'))
net.add(nn.Dense(10))
net.initialize(init.Normal(sigma=0.01))
```

One minor detail is of note when invoking `net.add()`. It adds one or more layers to the network. That is, an equivalent to the above lines would be `net.add(nn.Dense(256, activation='relu'), nn.Dense(10))`. Also note that Gluon automagically infers the missing parameteters, such as the fact that the second layer needs a matrix of size $256 \times 10$. This happens the first time the network is invoked.

`net.add()` 를 호출할 때 세밀하게 살펴봐야할 점이 있는데, 이 함수를 이용하면 한 개 이상의 레이어를 네트워크에 추가할 수 있다는 것입니다. 즉, 위 코드들로 정의된 뉴럴 네트워크는   `net.add(nn.Dense(256, activation='relu'), nn.Dense(10))` 코드 한 줄로 정의되 네트워크와 동일합니다. 또한, Gluon은 명시되지 않은 파라메터들을 자동으로 알아냅니다. 예를 들면, 두번째 레이어는  $256 \times 10​$ 크기의 행렬이 필요한데, 이는 네트워크가 처음 실행될 때 자동으로 찾아내 집니다.

We use almost the same steps for softmax regression training as we do for reading and training the model.

Softmax regssion 학습과 거의 같은 절차로 데이터를 읽고 모델을 학습 시킵니다.

```{.python .input  n=6}
batch_size = 256
train_iter, test_iter = d2l.load_data_fashion_mnist(batch_size)

loss = gloss.SoftmaxCrossEntropyLoss()
trainer = gluon.Trainer(net.collect_params(), 'sgd', {'learning_rate': 0.5})
num_epochs = 10
d2l.train_ch3(net, train_iter, test_iter, loss, num_epochs, batch_size, None,
              None, trainer)
```

## Problems

## 문제

1. Try adding a few more hidden layers to see how the result changes.
1. Try out different activation functions. Which ones work best?
1. Try out different initializations of the weights.
1. hidden 레이어들을 더 추가해서 결과가 어떻게 변하는지 확인하세요.
1. 다른 activation 함수를 적용해보세요. 어떤 것이 가장 좋게 나오나요?
1. weight에 대한 초기화를 다르게 해보세요.

## Scan the QR Code to [Discuss](https://discuss.mxnet.io/t/2340)

![](../img/qr_mlp-gluon.svg)
