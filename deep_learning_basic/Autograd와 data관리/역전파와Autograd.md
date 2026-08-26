# [역전파와 Autograd]

## 한 줄 정의
- (면접에서 바로 말할 수 있는 30초 정의)

## 왜 필요한가 / 어떤 문제를 해결하는가


## 핵심 동작 방식
![alt text](image.png)

## 예시 코드 또는 예시 상황

## 헷갈렸던 부분
- requires_grad=True가 leaf Tensor인지 아닌지 헷갈렸다
requires_grad=True는 해당 텐서의 gradient의 값이 정답에 어떤
영향을 줬는지 계산 과정을 기록해둔다고 했는데 leaf Tensor는 계산을 시작한
처음의 Tensor이므로 같지 않나라는 의문이 들었다
- .grad가 그냥 생기는 줄 알았는데 backward() 이후에 생기고 그 이전에는 None이다
- backward 이후 require_grad=True가 적용되어 있는 텐서의 그래디언트는 각 파라미터별로 저장되는지 몰랐다

> 1
# [개념/기술]이 내부적으로 동작하는 방식

## 궁금했던 질문
- 계산그래프가 뭔지?
- 체인룰은 뭐고 왜 사용하는지?
- requires_grad가 정확히 뭐고 leaf Tensor와 무슨 연관이 있는지?
- scalar loss를 만들기 위해 어떤 걸 해야하는지?
- require_grad를 하면 x.grad가 나오는 이유는?
- leaf Tensor의 gradient를 제외한 나머지 중간 Tensor의 gradient를 보고 싶다면 어떻게 해야하는지?
- with torch.no_grad()는 왜 사용하고 어디에 사용하는지?
- detach()와 clone()은 왜 사용하고 detach().clone()을 하면 어떤 효과가 생기는지?
- detach().clone()을 사용해서 뭘 확인하려고 하는지?
- detach와 no_grad()의 차이점은?
- print(param.shape, param.grad.shape)가 왜 같은지?

## 찾아본 내용 요약
- 계산 그래프는 Autograd가 "z가 x에서 어떤 연산들을 거쳐 만들어졌는가"를 backward에 적용하여 gradient를 계산하기 위해 기록하는 구조(순전파 때 만들어짐)
- 체인룰이란 여러 함수가 연결된 경우, 각 미분값을 곱해서 전체 미분값을 구하는 방법이며 gradient 계산에 사용함
- ∂L/∂w = ∂L/∂z ∂z/∂w 이처럼 w(가중치)에 대한 Loss값 즉 가중치의 변화에 따른 Loss값을 알기 위해 가중치 변화에 따른 선형변환 중간값 z와 그 중간값 z의 변화에 따른 Loss값을 곱하여 바로 목표하는 파라미터와 loss값을 계산하는게 아니라 체인룰에 의해 연결하는 방식을 backward는 자동으로 적용함
- requires_grad=True는 **해당** Tensor 연산을 추적해서 위와 같은 체인룰에 의한 gradient를 계산하도록 **설정**하는 것이고 보통 학습할 때 leaf Tensor(연산의 시작점)에 설정됨
- require_grad=True를 적용한 x가 계산 그래프의 leaf인 가정하에 backward()가 실행되면 ∂loss/∂x를 계산해서 x.grad에 저장
- backward()를 기본 방식으로 호출하려면 loss를 하나의 값(스칼라)로 만들어야 하므로 mean()이나 sum()을 이용함
- 중간 Tensor의 gradient를 확인하려면 해당 Tensor에 retain_grad()를 호출한 뒤 backward()하면 해당 Tensor의 .grad를 확인할 수 있음
- torch.no_grad(): gradient 계산/그래프 추적을 끄며, 추론·평가처럼 학습하지 않는 상황에서 학습한 모델을 평가하기 위해 gradient의 변화를 멈추고 메모리와 연산을 줄이기 위해 사용
- detach(): 기존 Tensor를 계산 그래프에서 분리 계산을 끊는 게 아니라, gradient의 역방향 전달을 끊는 것 
- clone(): 데이터를 복사해서 새로운 Tensor를 만듬
- detach().clone()은 그래프와 완전히 분리된 독립적인 복사본을 만듬
- detach().clone()으로 확인하는 것은 현재 Tensor의 값을 gradient 추적이나 원본 Tensor의 변경과 독립적으로 보관/비교하려고 사용
- detach() vs no_grad(): detach()는 **특정 Tensor**를 그래프에서 분리하고, no_grad()는 그 블록 안의 **연산 전체**에서 gradient 추적을 끈다.
- param.shape == param.grad.shape: gradient는 각 파라미터를 얼마나 바꿔야 하는지 나타내는 값이므로 해당 파라미터와 1:1로 대응해야 해서 shape이 같음
	​


## 그림/비유로 정리
- detach() ->
x
 ↓ ×3
z

z_detach
 ↓ ×2
y
 ↓ 제곱
loss에서 backward하면

loss
 ↓
y
 ↓
z_detach
 ✋ 이렇게 되고 그 이후의 파라미터인
x ← gradient 전달 안 됨
z ← gradient 전달 안 됨

## 결론
- gradient 계산을 검증하기 위해 필요한 도구
> 2

