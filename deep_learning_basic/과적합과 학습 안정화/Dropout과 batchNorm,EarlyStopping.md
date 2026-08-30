# [Dropout와 batchNorm, early stopping]

## 한 줄 정의
- 셋다 목적은 과적합을 방지하기 위해 사용되며 Dropout은 해당 데이터의 입력값을 선형변환을 통해 만들어진 특징들 중 일부 뉴런을 랜덤하게 꺼서 모델이 패턴을 외우는 것을 방지하였고 batchNorm은 말그대로 배치 기준으로 Normalization을 적용해서 각 Batch의 평균/분산으로 정규화하여 평가할 때 그 통계를 사용하는 것이고 Early Stopping은 Validation 성능이 더 이상 좋아지지 않으면 중단하는 방식을 사용

## 왜 필요한가 / 어떤 문제를 해결하는가
- 우선 Dropout은 일부 뉴런 즉 데이터의 입력값을 선형 변환을 통해 나온 출력값 중 일부를 0으로 만들어 모델이 지나치게 값을 외우지 않도록 하는 방법 중에 하나
- batchNorm은 1d,2d,3d까지 있으면 각각 차원에 맞게 사용, batchNorm은 데이터를 일정 수로 묶어 만든 배치 하나당 
주로 은닉층(Hidden Layer)의 Linear나 Conv 뒤에서 평균과 분산을 통해 정규화를 시킴 
이후에 평가할 때는 이렇게 각각 모은 평균과 분산을 평균 내어 running mean, running val로 만든 후 평가의 정규화에 적용함

Batch 1 → 평균 10 ─┐


Batch 2 → 평균 20 ─┤


Batch 3 → 평균 15 ─┤     → running mean


Batch 4 → 평균 25 ─┤

이는 과적합을 예방하기도 하지만 출력값들의 값의 차이를 줄여 학습의 안정화와 배치마다 값의 스케일이 크게 달라지는 문제를 완화함
- EarlyStopping은 학습 중 일정 주기(epoch)마다 성능을 확인하고 일정 기간 예를 들어 patience=3이라고 하면 마지막으로 좋아진
Loss값의 epoch 이후 3번의 epoch동안 Loss값이 min_delta=0.001보다 더 낮게 갱신하지 않으면 학습을 중단해서 과적합을 방지하는 방법

- 초기에는 어떤 모델이든 대부분 틀리기 때문에 반복하는 과정에서 오차를 줄일 수 있는 값을 제공하고자 함

## 핵심 동작 방식
![alt text](image.png)
![alt text](image-1.png)
![alt text](image-2.png)

## 헷갈렸던 부분
- dropout이 모든 layer에 각각 붙는 줄 알았는데 특정 히든 레이어만 적용한다는 사실을 알았음
- 배치가 데이터의 개수인줄 알았는데 여기서 말하는 배치는 데이터를 특정 개수씩 묶어 만든 데이터의 묶음 집합이라고 함
- 각 배치마다 병렬적으로 모델을 적용 시키는건줄 알았는데 그게 아니라 직렬적으로 배치 1이 끝나고 나면 해당 배치를 학습한 모델의 가중치를 배치 2에서 적용하여 사용하는 방식으로 사용함
- dropout은 모델을 만들 때 함께 넣어도 model.eval()에서는 자동으로 사용안된다는 사실을 몰랐다
- dropout과 BatchNorm은 model.train()과 model.eval()에서의 적용이 각각 다르며 EarlyStopping은 학습 과정에서만 사용 -> 평가에 적용하여 검사 받고 다음 epoch에서 반복

> 1
# [Dropout와 batchNorm, early stopping]가 내부적으로 동작하는 방식



## 궁금했던 질문


## 찾아본 내용 요약
- 

## 그림/비유로 정리
wnew​=w−η∂w∂L​

그래서 gradient는 방향과 변화율을 알려주고, learning rate와 optimizer가 실제 이동량을 결정한다.


## 결론
- 손실함수는 그래디언트를 적용한 옵티마이저와 학습률로 가중치를 옮기기 위해 필요한 오차를 계산하는 함수
> 2
