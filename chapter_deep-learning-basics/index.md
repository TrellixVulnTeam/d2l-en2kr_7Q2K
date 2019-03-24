# Deep Learning Basics

This chapter introduces the basics of deep learning. This includes network architectures, data, loss functions, optimization, and capacity control. In order to make things easier to grasp, we begin with very simple concepts, such as linear functions, linear regression, and stochastic gradient descent. This forms the basis for slightly more complex techniques such as the softmax (statisticians refer to it as multinomial regression) and multilayer perceptrons. At this point we are already able to design fairly powerful networks, albeit not with the requisite control and finesse. For this we need to understand the notion of capacity control, overfitting and underfitting. Regularization techniques such as dropout, numerical stability and initialization round out the presentation. Throughout, we focus on applying the models to real data, such as to give the reader a firm grasp not just of the concepts but also of the practice of using deep networks. Issues of performance, scalability and efficiency are relegated to the next chapters.

이 장에서는 딥 러닝의 기본적인 내용을들 소개합니다. 네트워크 아키텍처, 데이터, loss 함수, 최적화, 그리고 용량 제어를 포함합니다. 이해를 돕기 위해서, 선형 함수, 선형 회귀, 그리고 stochastic gradient descent 와 같은 간단한 개념부터 시작합니다. 이것들은 softmax나 multilayer perceptron와 같은 보다 복잡한 개념의 기초가 됩니다. 우리는 이미 상당히 강력한 네트워크를 디자인할 수 있지만, 필수적인 제어나 기교는 배우지 않습니다. 이를 위해서, 용량 제어, overfitting 과 underfitting에 대한 개념을 이해할 필요가 있습니다. Dropout, 수치 안정화(numerical stability), 그리고 초기화에 대한 설명으로 이 장을 마무리할 예정입니다. 우리는 실제 데이터에 모델을 적용하는 방법에 집중하겠습니다. 이를 통해서 여러분은 기본 개념 뿐만 아니라 딥 네트워크를 실제 문제에 적용할 수 있도록 할 예정입니다. 성능, 확장성 그리고 효율성은 다음 장들에서 다룹니다.

```eval_rst

.. toctree::
   :maxdepth: 2

   linear-regression
   linear-regression-scratch
   linear-regression-gluon
   softmax-regression
   fashion-mnist
   softmax-regression-scratch
   softmax-regression-gluon
   mlp
   mlp-scratch
   mlp-gluon
   underfit-overfit
   weight-decay
   dropout
   backprop
   numerical-stability-and-init
   environment
   kaggle-house-price
```
