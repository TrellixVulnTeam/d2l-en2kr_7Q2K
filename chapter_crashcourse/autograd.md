# Automatic Differentiation

# 자동 미분(automatic differentiation)

In machine learning, we *train* models to get better and better as a function of experience. Usually, *getting better* means minimizing a *loss function*, i.e. a score that answers "how *bad* is our model?" With neural networks, we choose loss functions to be differentiable with respect to our parameters. Put simply, this means that for each of the model's parameters, we can determine how much *increasing* or *decreasing* it might affect the loss. While the calculations are straightforward, for complex models, working it out by hand can be a pain (and often error-prone).

머신러닝에서 우리는 경험의 함수로 더욱 좋게 만들려는 목적으로 모델을 *학습* 시킵니다. 보통은 *더 좋게 만드는 것*은 *loss 함수* , 즉, "모델이 얼마나 *못하냐*" 에 대한 점수를 최소화하는 것을 의미합니다. 뉴럴 네트워크에서는 파라메터에 대해서 미분이 가능한 loss 함수를 선택합니다. 간단하게 설명하며, 모델의 각 파라메터들이 loss 를 얼마만큼 *증가* 또는 *감소* 시키는데 영향을 주는지를 결정할 수 있다는 것입니다. 미적분이 직관적이지만, 복잡한 모델들에 대한 미분을 직접 계산하는 것은 너무 힘들고 오류를 범할 수 있는 일입니다.

The autograd package expedites this work by automatically calculating derivatives. And while most other libraries require that we compile a symbolic graph to take automatic derivatives, `autograd` allows you to take derivatives while writing  ordinary imperative code. Every time you make pass through your model, `autograd` builds a graph on the fly, through which it can immediately backpropagate gradients. If you are unfamiliar with some of the math, e.g. gradients, please refer to the [“Mathematical Basics”](../chapter_appendix/math.md) section in the appendix.

`autograd` 패키지는 미분을 자동으로 계산해주는 기능 제공합니다. 다른 대부분의 라이브러리들은 자동 미분을 하기 위해서는 심볼 그래프를 컴파일을 우선 수행해야하지만, `autograd` 는 일반적인 imperative 코드를 작성하면서 미분을 수행할 수 있게 해줍니다. 모델에 값을 대입할 때 마다, `autograd` 는 그래프를 즉석으로 만들고, 이를 사용해서 gradient를 역으로 계산합니다. 이 절에서 나오는 수학에 익숙하지 않은 독자들은 부록의 [“Mathematical Basics”](../chapter_appendix/math.md) 절을 참고하세요.

```{.python .input  n=1}
from mxnet import autograd, nd
```

## A Simple Example

## 간단한 예제

As a toy example, let's say that we are interested in differentiating the mapping $y = 2\mathbf{x}^{\top}\mathbf{x}​$ with respect to the column vector $\mathbf{x}​$. Firstly, we create the variable `x` and assign an initial value.

장난감 예제로 시작해보겠습니다. 함수 $y = 2\mathbf{x}^{\top}\mathbf{x}$ 를 컬럼 백터 $\mathbf{x}$ 에 대해서 미분을 하고 싶습니다. 우선은, 변수 `x` 를 생성하고, 초기 값을 할당합니다.

```{.python .input  n=2}
x = nd.arange(4).reshape((4, 1))
print(x)
```

Once we compute the gradient of ``y`` with respect to ``x``, we'll need a place to store it. We can tell an NDArray that we plan to store a gradient by invoking its ``attach_grad()`` method.

`x` 에 대한 `y` 의 미분을 계산한 결과를 저장할 공간이 필요합니다. `attach_grad` 함수를 호출하면 NDArray는 gradient를 저장할 공간을 마련합니다.

```{.python .input  n=3}
x.attach_grad()
```

Now we're going to compute ``y`` and MXNet will generate a computation graph on the fly. It's as if MXNet turned on a recording device and captured the exact path by which each variable was generated.

자 이제 `y` 를 계산합니다. MXNet은 바로 연산 그래프를 생성해줍니다. 이는 마치 MXNet이 기록 장치를 키고, 각 변수들이 생성되는 정확한 경로를 모두 집어내는 것과 같습니다.

Note that building the computation graph requires a nontrivial amount of computation. So MXNet will *only* build the graph when explicitly told to do so. This happens by placing code inside a ``with autograd.record():`` block.

연산 그래프를 생성하는 데는 적지않은 연산량이 필요함을 기억해두세요. 따라서, MXNet은 *요청을 받았을 때만* 그래프를 생성하도록 되어 있습니다. 즉, 이는 `with autograd.record():` 블록 안에 넣었을 때 작동합니다.

```{.python .input  n=4}
with autograd.record():
    y = 2 * nd.dot(x.T, x)
print(y)
```

Since the shape of `x` is (4, 1), `y` is a scalar. Next, we can automatically find the gradient by calling the `backward` function. It should be noted that if `y` is not a scalar, MXNet will first sum the elements in `y` to get the new variable by default, and then find the gradient of the variable with respect to `x`.

`x` 의 shape이 (4,1) 이니, `y` 는 스칼라 값이 됩니다. 이후에 우리는 `backward` 함수를 호출해서 gradient를 자동으로 찾을 구합니다. 만약 `y` 가 스칼라 타입이 아니면, 기본 설정으로 MXNet은 우선 `y` 의 모든 항목을 더해서 새로운 값을 얻은 결과의  `x` 에 대한 gradient를 찾습니다.

```{.python .input  n=5}
y.backward()
```

The gradient of the function $y = 2\mathbf{x}^{\top}\mathbf{x}$ with respect to $\mathbf{x}$ should be $4\mathbf{x}$. Now let's verify that the gradient produced is correct.

함수  $y = 2\mathbf{x}^{\top}\mathbf{x}$ 의  $\mathbf{x}$ 에 대한 미분은  $4\mathbf{x}$ 입니다. 미분이 올바르게 계산되는지 확인해보겠습니다.

```{.python .input  n=6}
print((x.grad - 4 * x).norm().asscalar() == 0)
print(x.grad)
```

## Training Mode and Prediction Mode

## 학습 모드와 예측 모드

As you can see from the above, after calling the `record` function, MXNet will record and calculate the gradient. In addition, `autograd` will also change the running mode from the prediction mode to the training mode by default. This can be viewed by calling the `is_training` function.

위에서 보았듯이 `record` 함수를 호출하면 MXNet은 gradient를 계산하고 기록합니다. 더불어 기본 설정으로는 `autograd` 는 예측 모드에서 학습 모드로 실행 모드를 바꿉니다. 이는 `is_training` 함수를 호출해서 확인할 수 있습니다.

```{.python .input  n=7}
print(autograd.is_training())
with autograd.record():
    print(autograd.is_training())
```

In some cases, the same model behaves differently in the training and prediction modes (such as batch normalization). In other cases, some models may store more auxiliary variables to make computing gradients easier. We will cover these differences in detail in later chapters. For now, you need not worry about these details just yet.

어떤 경우에는 같은 모델이 학습 모드와 예측 모드에서 다르게 동작합니다. 배치 normalization이 그렇습니다. 다른 어떤 모델은 gradient를 쉽게 계산하기 위해서 더 많은 보조 변수를 저장하기도 합니다. 다음 장들에서 차이에 대해서 자세하게 다룰 예정이나, 지금은 자세한 내용에 대해서 걱정할 필요는 없습니다.

## Computing the Gradient of Python Control Flow

## Python 제어 프름의 gradient 계산하기

One benefit of using automatic differentiation is that even if the computational graph of the function contains Python's control flow (such as conditional and loop control), we may still be able to find the gradient of a variable. Consider the following program:  It should be emphasized that the number of iterations of the loop (while loop) and the execution of the conditional judgment (if statement) depend on the value of the input `b`.

자동 미분을 사용하는 장점 중에 하나는 함수의 연산 그래프가 Python 제어 흐름(조건과 loop 제어)을 가지고 있어도 변수에 대한 gradient를 찾아낼 수 있다는 것입니다. 아래 프로그램을 예로 들어보겠습니다. while loop의 반복 횟수와 if 조건문의 수행 횟수는 입력 `b` 의 값에 영향을 받습니다.

```{.python .input  n=8}
def f(a):
    b = a * 2
    while b.norm().asscalar() < 1000:
        b = b * 2
    if b.sum().asscalar() > 0:
        c = b
    else:
        c = 100 * b
    return c
```

Note that the number of iterations of the while loop and the execution of the conditional statement (if then else) depend on the value of `a`. To compute gradients, we need to `record` the calculation, and call the `backward` function to find the gradient.

Gradient를 계산하기 위해서 `record` 함수를 호출해야 하고, gradient를 찾기 위해서 `backward` 함수를 호출합니다.

```{.python .input  n=9}
a = nd.random.normal(shape=1)
a.attach_grad()
with autograd.record():
    d = f(a)
d.backward()
```

Let's analyze the `f` function defined above. As you can see, it is piecewise linear in its input `a`. In other words, for any `a` there exists some constant such that for a given range `f(a) = g * a`. Consequently `d / a` allows us to verify that the gradient is correct:

위에서 정의한 함수 `f` 를 분석해보겠습니다. 실제로, 임의의 입력 `a`가 주어지면, 그 출력은 `f (a) = x * a`의 형태여야 하며, 여기서 스칼라 계수 `x`의 값은 입력 `a`에 의존합니다. `c = f (a)`는 `a`에 대한 `x`의 기울기와 `c / a`의 값을 갖기 때문에 다음과 같이 이 예에서 제어 흐름의 gradient 결과의 정확성을 확인할 수 있습니다.

```{.python .input  n=10}
print(a.grad == (d / a))
```

## Head gradients and the chain rule

## Head gradient와 체인룰

*Caution: This part is tricky and not necessary to understanding subsequent sections. That said, it is needed if you want to build new layers from scratch. You can skip this on a first read.*

*주의: 이 파트는 까다롭지만, 이 책의 나머지 부분을 이해하는데 꼭 필요한 내용은 아닙니다. 만약 새로운 네트워크를 직접 만들고 싶은 경우 필요한 내용이기 때문에, 이 책을 처음 읽을 때는 넘어가도 됩니다.

Sometimes when we call the backward method, e.g. `y.backward()`, where
`y` is a function of `x` we are just interested in the derivative of
`y` with respect to `x`. Mathematicians write this as
$\frac{dy(x)}{dx}​$. At other times, we may be interested in the
gradient of `z` with respect to `x`, where `z` is a function of `y`,
which in turn, is a function of `x`. That is, we are interested in
$\frac{d}{dx} z(y(x))​$. Recall that by the chain rule

`y` 가 `x`  의 함수이고, 이 함수의 backward 함수, `y.backward()` 를 부를 때, 우리는 `y` 의 `x` 에 대한 미분을 구하는 것이 관심이 있습니다. 수학자들은 이를 $\frac{dy(x)}{dx}$ 로 기술합니다. 어떤 경우에는 `z` 가 `y` 의 함수일 때, `z` 의 `x` 에 대한 미분을 구해야 합니다. 즉, $\frac{d}{dx} z(y(x))$ 를 구햐야 합니다. 체인 규칙을 떠올려보면, 다음과 같이 계산됩니다.

$$\frac{d}{dx} z(y(x)) = \frac{dz(y)}{dy} \frac{dy(x)}{dx}.$$

So, when ``y`` is part of a larger function ``z``, and we want ``x.grad`` to store $\frac{dz}{dx}​$, we can pass in the *head gradient* $\frac{dz}{dy}​$ as an input to ``backward()``. The default argument is ``nd.ones_like(y)``. See [Wikipedia](https://en.wikipedia.org/wiki/Chain_rule) for more details.

`y` 가 더 큰 함수 `z` 의 일부이고, `x.grad` 가 $\frac{dz}{dx}$ 를 저장하도록 하려면,  *head gradient* $\frac{dz}{dy}$ 를 `backward()` 의 입력으로 넣으면 됩니다. 기본 인자는 ``nd.ones_like(y)`` 입니다. 자세한 내용은  [Wikipedia](https://en.wikipedia.org/wiki/Chain_rule) 를 참조하세요.

```{.python .input  n=11}
with autograd.record():
    y = x * 2
    z = y * x

head_gradient = nd.array([10, 1., .1, .01])
z.backward(head_gradient)
print(x.grad)
```

## Summary

## 요약

* MXNet provides an `autograd` package to automate the derivation process.
* MXNet's `autograd` package can be used to derive general imperative programs.
* The running modes of MXNet include the training mode and the prediction mode. We can determine the running mode by `autograd.is_training()`.
* MXNet은 미분 계산을 자동화해주는 `autograd` 패키지를 제공합니다.
* MXNet의 `autograd` 패키지는 일반적인 imperative 프로그램 형식으로 사용할 수 있습니다.
* MXNet의 실행 모드는 학습 모드와 예측 모드가 있습니다. 현재의 실행 모드는 `autograd.is_training()` 을 통해서 구할 수 있습니다.

## Problems

## 문제

1. In the example, finding the gradient of the control flow shown in this section, the variable `a` is changed to a random vector or matrix. At this point, the result of the calculation `c` is no longer a scalar. What happens to the result. How do we analyze this?
1. Redesign an example of finding the gradient of the control flow. Run and analyze the result.
1. In a second price auction (such as in eBay or in computational advertising) the winning bidder pays the second highest price. Compute the gradient of the winning bidder with regard to his bid using `autograd`. Why do you get a pathological result? What does this tell us about the mechanism? For more details read the paper by [Edelman, Ostrovski and Schwartz, 2005](https://www.benedelman.org/publications/gsp-060801.pdf).
1. Why is the second derivative much more expensive to compute than the first derivative?
1. Derive the head gradient relationship for the chain rule. If you get stuck, use the  [Wikipedia Chain Rule](https://en.wikipedia.org/wiki/Chain_rule) entry.
1. Assume $f(x) = \sin(x)$. Plot $f(x)$ and $\frac{df(x)}{dx}$ on a graph, where you computed the latter without any symbolic calculations, i.e. without exploiting that $f'(x) = \cos(x)$.
1. 이 절에서 예를 들었던 control flow의 gradient를 찾고자 할 때, 변수 `a` 가 랜덤 백터 또는 행렬에 따라서 바뀌고 있습니다. 이 때, `c` 계산 결과는 더 이상 스칼라 형태가 아닙니다. 결과에 무슨 일이 일어났나요? 이것을 어떻게 분석할 수 있나요?
1. control flow의 gradient를 구하는 예를 다시 설계해보세요. 실행하고 결과를 분석해보세요.
1. 세컨드 가격 경매 (예, eBay)에서는 두번째로 높은 가격을 부른 사람이 물건을 사게 됩니다. `autograd` 를 이용해서 물건의 가격에 대해서 경매 참여에 대한 gradient를 계산하세요. 왜 병리학적인 결과가 나올까요? 이 결과는 메카니즘에 대해서 무엇을 말해주나요? 더 자세한 내용은 [Edelman, Ostrovski and Schwartz, 2005](https://www.benedelman.org/publications/gsp-060801.pdf) 페이퍼를 읽어보세요.
1. 왜 일차미분보다 이차미분이 훨씬 더 많은 연산량이 필요하나요?
1. 체인룰의 head gradient 관계를 유도해보세요. 막히는 경우 [Wikipedia Chain Rule](https://en.wikipedia.org/wiki/Chain_rule) 를 참고하세요.
1. 함수 $f(x) = \sin(x)$ 를 가정합니다. 심볼 계산을 하지 않고 (즉, $f'(x) = \cos(x)$을 이용하지 말고) $\frac{df(x)}{dx}$ 를 구해서 $f(x)$ 와 $\frac{df(x)}{dx}$ 의 그래프를 그려보세요.

## Scan the QR Code to [Discuss](https://discuss.mxnet.io/t/2318)

![](../img/qr_autograd.svg)
