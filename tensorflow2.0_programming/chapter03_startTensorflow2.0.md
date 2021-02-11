# chapter03 텐서플로2.0 시작하기
~~~python
# tensorflow 2.0 버전 선택
try:
  # %tensorflow_version only exists in Colab
  %tensorflow_version 2.x
except Exception:
  pass
import tensorflow as tf
~~~
- 퍼센트 기호(%) 붙은 명령은 매직 커맨드라고 함. 이것은 구글 코랩의 원조격인 인터렉티프 파이썬(IPython)부터 지원하는 명령으로, 여러가지 기능을 미리 정의해놓아서 편리하게 사용할 수 있음
- 신경망의 초깃값을 지정해주는 것을 초기화(initialization)라고 하며, 이와 관련해서도 많은 논문들이 나와 있는데, 현재 가장 많이 사용하는 것은 Xavier 초기화와 He 초기화임
- Xavier 초기화와 He 초기화는 균일 분포나 정규 분포 중 하나를 택해서 신경망의 초깃값을 초기화함
- 다음은 단순하게 난수를 만들어보자
~~~python
rand = tf.random.uniform([1], 0, 1)
print(rand)

# 결과
tf.Tensor([0.26958692], shape=(1,), dtype=float32)
~~~
- shape은 행렬을 구성하는 행, 열 등 차원의 수를 나타내는 값
- shape을 직접 세는 방법은 바깥쪽부터 안쪽으로 원소의 개수를 세는 것   
  [[2], [3]]의 경우 바깥쪽은 원소의 개수가 [2], [3]으로 2개이고, 안쪽으로 들어가면 한개씩 있기 때문에 shape=[2, 1]이 됨
- dtype은 data type을 의미하며 float32는 32비트의 부동소수점 수라는 뜻
- 텐서플로 2.0에서는 16비트 부동소수점 수인 float16과 64비트 부동소수점 수인 float64도 존재
- shape의 개수를 바꿔서 여러개의 난수를 얻을 수 있음
~~~python
rand = tf.random.uniform([4], 0, 1)

# 결과
tf.Tensor([0.95959496 0.18216872 0.06052613 0.28986788], shape=(4,), dtype=float32)
~~~
- 균일 분포 외에 정규 분포로도 난수를 생성할 수 있음. 평균과 표준편차 값을 파라미터로 입력받음
~~~python
rand = tf.random.normal([4], 0, 1)

# 결과
tf.Tensor([1.5549321  2.70452    0.2689144  0.47025424], shape=(4,), dtype=float32)
~~~ 

## 뉴런 만들기
- 뉴런을 퍼셉트론이라고도 하며 입력을 받아 출력을 반환하는 단순한 구조
- 뉴런은 입력, 가중치, 활성화함수, 출력으로 구성됨
- 입력, 가중치, 출력은 보통 정수(integer)나 앞에서 살펴본 float이 많이 쓰임
- 신경망 초창기에는 시그모이드를 주로 사용했지만, 은닉층을 다수 사용하는 딥러닝 시대로 넘어오면서 ReLu가 더 많이 쓰이고 있음
- 딥러닝에서 오류를 backpropagate 할 때 시그모이드 함수가 값을 점점 작아지게 만드는 문제를 토론토 대학의 비노드 네어와 제프리 힌튼 교수가 지적하며 ReLU를 대안으로 제시
- ReLU는 양수를 그대로 반환하기 때문에 값의 왜곡이 적어짐
- 다음은 시그모이드 함수를 파이썬으로 구현해 보자
~~~python
import maths
def sigmoid(x):
    return 1 / (1 + math.exp(-x))
~~~
- 다음은 뉴런의 입력과 출력을 정의해보자  
  입력이 1일 때 기대출력이 0이 되는 뉴런을 만들어보자
~~~python
x = 1
y = 0
w = tf.random.normal([1], 0, 1)
output = sigmoid(x * w)
print(output)
~~~
- 뉴런이란 결국 가중치(w) 값을 의미함. 이 w의 값을 변화시켜야 하는데 경사하강법을 이용해서 개선시킴
- 이것은 w에 입력과 학습률과 에러를 곱해준 값을 더해주는 것  
  W = W + X * learningRate * error
- 경사 하강법이 효과를 발휘하는지 다음의 코드를 통해서 확인해보자
~~~python
for i in range(1000):
  output = sigmoid(x * w)
  error = y - output
  w = w + x * 0.1 * error

  if i % 100 == 99:
    print(i, error, output)

# 결과 확인
99 -0.1151402194478985 0.1151402194478985
199 -0.05576779980642165 0.05576779980642165
299 -0.036357323077621094 0.036357323077621094
399 -0.02687599574220274 0.02687599574220274
499 -0.02128536001316444 0.02128536001316444
599 -0.01760667392078541 0.01760667392078541
699 -0.015005505517835086 0.015005505517835086
799 -0.013070348048286436 0.013070348048286436
899 -0.01157512023978612 0.01157512023978612
999 -0.010385510603554667 0.010385510603554667
~~~
- 다음과 같은 식에서 x = 0이고 y = 1을 얻는 뉴런은 구할 수 없음  
  why? w = x * w * error의 식에서 x = 0이기 때문에 w에 더해지는 값은 없음
- 이러한 현상 때문에 편향(bias)를 넣어줌. 편향은 w처럼 난수로 초기화되며 뉴런에 더해져서 출력을 계산하게 됨 
- 수식에서는 관용적으로 bias의 앞 글자인 b를 씀   
  Y = f(X*w + 1*b)
- 그럼 실제 코드로 확인해보자
~~~python
x = 0
y = 1
w = tf.random.normal([1], 0, 1)
b = tf.random.normal([1], 0, 1)

for i in range(1000):
    output = sigmoid(x * w + 1 * b)
    error = y - output
    w = w + x * 0.1 * error
    b = b + 1 * 0.1 * error

    if i % 100 == 99:
        print(i, error, output)

# 결과
99 0.08765619395480928 0.9123438060451907
199 0.04808745028837935 0.9519125497116206
299 0.032874934971591996 0.967125065028408
399 0.02490876742537529 0.9750912325746247
499 0.020026323368105725 0.9799736766318943
599 0.016733527477148624 0.9832664725228514
699 0.014365163517259583 0.9856348364827404
799 0.012580950075645858 0.9874190499243541
899 0.01118914141658156 0.9888108585834184
999 0.01007339246062633 0.9899266075393737
~~~

## 3.3.3 첫번째 신경망 네트워크: AND
- 이 네트워크의 구성요소는 앞에서 본 것과 같은 하나의 뉴런임
- 여기서는 AND 연산을 수행하는 뉴런을 만들어보겠는데, AND 연산은 여러개의 입력을 받을 수 있지만 여기서는 2개의 입력만으로 제한하겠음
- 다음과 같은 식으로 표현할 수 있음  
  Y = f(X1*w1 + X2*w2 + 1*b)
- 입력은 여러개로 표현할 수 있지만, 보통 편향은 1개만 사용
- 실제 코드로 확인해보자
~~~python
import numpy as np
x = np.array([[1,1], [1,0], [0,1], [0,0]])
y = np.array([[1], [0], [0], [0]])
w = tf.random.normal([2], 0, 1)
b = tf.random.normal([1], 0, 1)
b_x = 1

for i in range(2000):
  error_sum = 0
  for j in range(4):
    output = sigmoid(np.sum(x[j]*w) + b_x * b)
    error = y[j][0] - output
    w = w + x[j] * 0.1 * error
    b = b + b_x * 0.1 * error
    error_sum += error

  if i % 200 == 199:
    print(i, error_sum)

# 결과
199 -0.09654696319636714
399 -0.06057416175452414
599 -0.0439935744906196
799 -0.034447007124115164
999 -0.0282577146082114
1199 -0.023928386685121053
1399 -0.020735308895332434
1599 -0.018284656291288618
1799 -0.016347855943776173
1999 -0.01477725368321514    
~~~
- 넘파이는 수학과 과학 연산에 특화된 파이썬 모듈로, 딥러닝에서도 유용하게 사용됨
- x와 y를 넘파이 array로 정의함. 넘파이 array는 파이썬의 리스트보다 적은 메모리를 차지하며 계산 속도가 빠름
- 넘파이에 실수를 곱하면 array의 각 원소에 대해 자동으로 실수를 곱하는 연산이 이루어짐  
  이를 <b>각 원소에 대한(element-wise) 연산</b>이라고 함
~~~python
import numpy as np
print(np.array([1,2,3]) * 3)

# 결과
[3 6 9]
~~~
- 결괏값을 자세히 보면 쉼표(,)가 없는 것도 확인 가능. 각 원소에 대한 연산으로 나온 결괏값은 파이썬의 리스트가 아닌 넘파이 array로 자동 변환되기 때문
- 이렇게 학습시킨 네트워크가 정상적으로 작동하는지 평가해보자
~~~python
for i in range(4):
  print('X: ', x[i], 'Y: ', y[i], 'Output:', sigmoid(np.sum(x[i]*w)+b))

# 결과
X:  [1 1] Y:  [1] Output: 0.9655042629975893
X:  [1 0] Y:  [0] Output: 0.024449155317244647
X:  [0 1] Y:  [0] Output: 0.0245228139610779
X:  [0 0] Y:  [0] Output: 2.2509631548148593e-05
~~~

## 3.3.4 두 번째 신경망 네트워크 : OR
- OR 연산을 계산하는 네트워크를 생성하는 코드도 AND 네트워크와 매우 비슷
- 달라진 것은 y 부분의 기대출력
~~~python
import numpy as np
x = np.array([[1,1], [1,0], [0,1], [0,0]])
y = np.array([[1], [1], [1], [0]])
w = tf.random.normal([2], 0, 1)
b = tf.random.normal([1], 0, 1)
b_x = 1

for i in range(2000):
  error_sum = 0
  for j in range(4):
    output = sigmoid(np.sum(x[j]*w) + b_x * b)
    error = y[j][0] - output
    w = w + x[j] * 0.1 * error
    b = b + b_x * 0.1 * error
    error_sum += error

  if i % 200 == 199:
    print(i, error_sum)

# 결과
199 -0.05160559063311346
399 -0.02642701666431979
599 -0.017648981405738146
799 -0.013200749871762202
999 -0.010523483886790738
1199 -0.008739831817495207
1399 -0.007468405835016692
1599 -0.006516717730039048
1799 -0.00577905986975135
1999 -0.005188935960780201

for i in range(4):
  print('X: ', x[i], 'Y: ', y[i], 'Output:', sigmoid(np.sum(x[i]*w)+b))

X:  [1 1] Y:  [1] Output: 0.9999971407268853
X:  [1 0] Y:  [1] Output: 0.9897042189046867
X:  [0 1] Y:  [1] Output: 0.9896977643309127
X:  [0 0] Y:  [0] Output: 0.025725016614416278
~~~

## 3.3.5 세 번째 신경망 네트워크: XOR
- XOR은 홀수 개의 입력이 참일때만 참
- 인공지능의 겨울을 불러왔던 XOR
- AND, OR 네트워크와 똑같이 작성하면 어떻게 될까?
~~~python
import numpy as np
x = np.array([[1,1], [1,0], [0,1], [0,0]])
y = np.array([[0], [1], [1], [0]])
w = tf.random.normal([2], 0, 1)
b = tf.random.normal([1], 0, 1)
b_x = 1

for i in range(2000):
  error_sum = 0
  for j in range(4):
    output = sigmoid(np.sum(x[j]*w) + b_x * b)
    error = y[j][0] - output
    w = w + x[j] * 0.1 * error
    b = b + b_x * 0.1 * error
    error_sum += error

  if i % 200 == 199:
    print(i, error_sum)

# 결과
199 0.00036327791831713974
399 1.4771347023101455e-05
599 6.012390765253173e-07
799 2.792131748030613e-09
999 1.8614210173240053e-09
1199 1.8614210173240053e-09
1399 1.8614210173240053e-09
1599 1.8614210173240053e-09
1799 1.8614210173240053e-09
1999 1.8614210173240053e-09
~~~
- 에러 값이 어느 순간 변하지 않음. 이렇게 학습시킨 네트워크를 평가해보면 결과는 다음과 같음
~~~python
for i in range(4):
  print('X: ', x[i], 'Y: ', y[i], 'Output:', sigmoid(np.sum(x[i]*w)+b))

X:  [1 1] Y:  [0] Output: 0.5128176323940516
X:  [1 0] Y:  [1] Output: 0.5128176314633411
X:  [0 1] Y:  [1] Output: 0.4999999990686774
X:  [0 0] Y:  [0] Output: 0.49999999813735485
~~~
- 어떻게 해서 이런 결과가 나오는 것일까? 
- 시그모이드 함수를 통과시키기 전 값이 0에 가깝기 때문에 최종 출력이 0.5에 가깝게 나옴
- AND 네트워크의 가중치를 보면 두 개의 가중치가 비슷하면서 절대값이 큼(6.94, 6.95)  
  또한 편향 값이 -10.6으로 나온 것을 확인
- 반면에 XOR 네트워크의 가중치 값은 우선 값이 너무 작음. 즉 XOR 네트워크는 어떤 일을 하려고 하는지 명확하지가 않음
- 이것이 바로 XOR(XOR Problem)임. 즉 하나의 퍼셉트론으로는 간단한 XOR 연산자도 만들어낼 수 없음을 <퍼셉트론> 서적에서 말하고 있음
- 이러한 문제의 해결책은 여러 개의 퍼셉트론을 사용하는 것  
- 여기서 3개의 퍼셉트론과 뉴런을 사용해 보겠음. 코드가 복잡해 지는 것을 막기 위해 1.2.4절에서 설명한 tf.keras를 사용하겠음. 
~~~python
import numpy as np
x = np.array([[1,1], [1,0], [0,1], [0,0]])
y = np.array([[0], [1], [1], [0]])

model = tf.keras.Sequential([
    tf.keras.layers.Dense(units = 2, activation='sigmoid', input_shape = (2,)),
    tf.keras.layers.Dense(units = 1, activation='sigmoid')
])

model.compile(optimizer=tf.keras.optimizers.SGD(lr=0.1), loss = 'mse')
model.summary()
~~~
- `tf.keras`는 딥러닝 계산을 편하게 하기 위한 추상적인 클래스
- 쉽게 말해서 딥러닝 계산을 위하여 여러 함수와 변수의 묶음이라고 생각하면 됨
- 모델에서 가장 많이 쓰이는 구조가 `tf.keras.Sequential` 구조임  
  뉴런과 뉴런이 합쳐진 단위인 레이어를 일직선으로 배치한 것
- `tf.keras.layers.Dense`는 model에서 사용하는 레이어를 정의하는 명령
- 레이어의 입력과 출력 사이에 있는 모든 뉴런이 서로 연결되어 있는 레이어 
- `tf.keras.layers.Dense` 안의 units는 레이어 안에 있는 뉴런의 수를 뜻함  
  뉴런이 많아질수록 레이어의 성능은 좋아지지만 계산량도 많아지고, 메모리도 많이 차지하게 됨
- `input_shape`는 시퀀셜 모델의 첫 번째 레이어에서만 정의하는데, 입력의 차원 수가 어떻게 되는지를 정의함
- 이렇게 정의된 2-레이어 XOR 네트워크 구조를 그림으로 나타내면 다음과 같음

![img](2_Layer_XORNetwork.JPG)

- 네트워크 구조에서 실선으로 그려지는 화살표는 가중치를 나타냄
- 그런데 [OUT]에 표시된 콘솔 출력 결과에서 Param을 보면 첫 번째 레이어에서는 6개, 두 번째 레이어에서는 3개의 파라미터가 있다고 나옴
- 이것은 바로 각 레이어가 기본적으로 편향을 포함하고 있기 때문 
- 편향을 포함한 네트워크 구조를 다시 그리면 다음과 같음

![img](biased_2_Layer_XORNetwork.JPG)

- 보통 Dense 레이어의 파라미터 수는 (입력측 뉴런의 수 + 1) x (출력측 뉴런의 수)의 식으로 구할 수 있음
- 여기서 말하는 입력측은 Dense 레이어에 들어오는 입력을 입력측, Dense 레이어의 뉴런을 출력측이라고 함
- 이 식에 따르면 첫 번째 레이어 수는 (2+1) * 2 = 7, 두 번째 레이어 수 (2+1) * 1 = 3 임
- 이제는 네트워크를 실제로 학습시켜볼 차례
~~~python
history = model.fit(x, y, epochs = 2000, batch_size = 1)

# 결과 
Train on 4 samples
Epoch 1/2000
4/4 [==============================] - 0s 75ms/sample - loss: 0.2574
Epoch 2/2000
4/4 [==============================] - 0s 3ms/sample - loss: 0.2572
Epoch 3/2000
4/4 [==============================] - 0s 2ms/sample - loss: 0.2570
Epoch 4/2000
4/4 [==============================] - 0s 2ms/sample - loss: 0.2567
Epoch 5/2000
4/4 [==============================] - 0s 2ms/sample - loss: 0.2566
...
~~~
- `model.predict()` 함수에 입력을 넣으면 네트워크 출력 결과를 알 수 있음
- 결과적으로 잘 예측한 것을 확인할 수 있는데, 처음의 XOR 네트워크와 비교했을 때 어떻게 잘 예측할 수 있었을까?
- 일단 가중치와 편향을 출력해보자
~~~python
for weight in model.weights:
  print(weight)
~~~
- 가중치 정보는 `model.weight`에 저장되어 있음
- 입력과 레이어, 레이어와 레이어 사이의 뉴런을 연결할 때 사용되는 가중치는 <b>kernel</b>이고, 편향과 연결된 가중치는 bias로 표시됨
- 보통 네트워크의 가중치 숫자가 많기 때문에 편의상 가중치에 첨자를 붙여 표시
- 레이어의 윗첨자는 순서를 나타내며, 아래첨자는 뉴런의 순서에 맞게 차례로 붙임
- 앞에서본 1개의 뉴런, 레이어를 사용한 XOR, AND와는 다르게 뉴런 개수가 3개, 레이어 개수가 2개로 늘면서 이 가중치들이 정확히 어떤 일을 하는지 한눈에 잘 들어오지 않음
- 즉 가중치 시각화보다 네트워크의 학습 상황을 더 잘 파악할 수 있는 방법이 필요

## 3.4 시각화 기초
- 파이썬에서 시각화하는 방법 중 대표적인 것을  `matplotlib.pyplot`을 이용한 그래프 그리기가 있음

### 3.4.1. matplotlib.pyplot을 이용한 그래프 그리기
- 먼저 그래프에 익숙해져보자  
  꺾은선 그래프를 그려보자
~~~python
import matplotlib.pyplot as plt
x = range(20)
y = tf.random.normal([20], 0, 1)
plt.plot(x, y)
plt.show()
~~~
- 보통 `matplotlib.pyplot`은 plt로 축약
- 이렇게 그려진 그래프는 `plt.show()`라는 함수를 호출해야만 주피터 노트북이나 코랩에서 확인 가능
- 점으로 바꾸려면 `plt.plot(x, y, 'bo')` 로 변경
- 여기서 추가된 'bo' 부분은 파란색(blue) 점(o)를 나타냄  
  선을 나타내고 싶으면 'b-' , 점선을 나타내고 싶으면 'b--', 색깔을 나타내고 싶으면 b를 다른색으로 변경
- 다음과 같이 히스토그램도 그릴 수 있음
~~~python
import matplotlib.pyplot as plt
random_normal = tf.random.normal([100000], 0, 1)
plt.hist(random_normal, bins = 100)
plt.show() 
~~~
- `bins`는 histogram의 구간을 의미함

## 3.4.2 2-레이어 XOR 네트워크의 정보 시각화
- 다음과 같이 `model.fit` 실행결과 객체를 return 받아 해당 객체의 loss 값을 시각화 할 수 있음
~~~python
plt.plot(history.history['loss'])
~~~
- `plt.plot`에 하나의 변수만 전달하면 그 변수를 y로 간주하고, x는 자동으로 `range(len(y))`에 해당하는 값을 넣어 그래프를 만들어줌