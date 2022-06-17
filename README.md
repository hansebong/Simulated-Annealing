# Simulated-Annealing
## 모의 담금질 기법이란??
모의 담금질 기법은 높은 온도에서 액체 상태인 물질이 온도가 점차 낮아지면서 결정체로 변하는 과정을 모방한 해 탐색 알고리즘이라고 할 수 있다.  
담금질 기법이라는 말은 금속 공학의 담금질에서 왔는데, 금속 공학의 담금질이 금속재료를 가열한 다음 조금씩 냉각하여 결정을 성장시켜서 결함을 줄이는 작업인것 처럼 이 알고리즘도 해를 반복해 개선함으로써 현재의 해 근방에 있는 해를 임의로 찾는다. 이때 알고리즘에서 온도라는 전역인자가 영향을 주는데, 처음에는 온도가 높기 때문에 해가 크게 변환 하지만, 온도가 0에 가까워짐에 따라서 변화는 줄어든다.

## 회귀식 추정
1. 우선 회귀식 추정을 하기 전에 하나의 독립변수와 종속변수로 이루어진 데이터 집합을 구해야 한다.나는 이러한 데이터의 집합의 예시로 미국의 1인당 실질 GDP를 선정하였다.  

![image](https://m1.daumcdn.net/cfile249/image/254BC64D57484671149AF0)  

2. 위의 그림에서 대략 수학적 모형을 가정해 보았는데, y가 1인당 실질 GDP라고 가정하고 x를 1975년부터 지난년도 수라고  가정을 해보았을 때 식은 y=571x+30000이다.  
이 그래프와 식으로 미국의 1인당 실질 GDP가 1975년부터 2015년까지 꾸준히 우상향 한다는 것을 알 수 있다.  
이때 1인당 실질 GDP는 종속변수이고 연도수는 독립변수가 된다.

## 모의 담금질의 작동방식
 모수값 추정을 하기 전에 모의 담금질 기법이 어떻게 돌아가는지에 대해서 설명하고자 한다.  
![image](https://t1.daumcdn.net/cfile/tistory/2501553C518D326902)  
위의 그림은 모의 담금질의 수도코드이다, 이러한 방식으로 모의 담금질은 돌아가는데, 여기서 Schedule[t]라는 것은 Temperature를 정해주는 함수이다. 그리고 VALUE[] 라는 것은 평가 함수의 값이다. 그리고 MAKE-NODE()는 노드를 만들어 주는 함수라고 한다.  
글로만 써 보았을 때에는 설명하기가 조금 어려워 가져온 코드를 가지고 설명을 하고자 한다. 
~~~
 static public void Main (){
        // Problem: place k objects in an MxN target
        // plane yielding minimal cost according to
        // defined objective function
  
        // Set of all possible candidate locations
        String[,] sourceArray = new String[M,N];
  
        // Global minimum
        Solution min = new Solution(Double.MaxValue, null);
  
        // Generates random initial candidate solution
        // before annealing process
        Solution currentSol = genRandSol();
  
        // Continues annealing until reaching minimum
        // temperature
        while (T > Tmin) {
            for (int i=0;i<numIterations;i++){
  
                // Reassigns global minimum accordingly
                if (currentSol.CVRMSE < min.CVRMSE){
                    min = currentSol;
                }
  
                Solution newSol = neighbor(currentSol);
                double ap = Math.Pow(Math.E,(currentSol.CVRMSE - newSol.CVRMSE)/T);
                Random rnd = new Random();
                if (ap > rnd.Next(0,1)){
                    currentSol = newSol;
                }  
            }
  
            T *= alpha; // Decreases T, cooling phase
        }
~~~
C#으로 이루어져 있어서 조금 헷갈렸지만 아무튼 위에서 주어진 수도코드와 비슷하게 작동을 한다.  
~~~
  while (T > Tmin) {
            for (int i=0;i<numIterations;i++){
  
                // Reassigns global minimum accordingly
                if (currentSol.CVRMSE < min.CVRMSE){
                    min = currentSol;
                }
  
                Solution newSol = neighbor(currentSol);
                double ap = Math.Pow(Math.E,(currentSol.CVRMSE - newSol.CVRMSE)/T);
                Random rnd = new Random();
                if (ap > rnd.Next(0,1)){
                    currentSol = newSol;
                }  
            }
  
            T *= alpha; // Decreases T, cooling phase
        }
~~~  
이 부분에서 초기 T가 종료 조건이 만족될때까지 for문이 실행되고, 이 for문에서 numIterations은 현재 T에서 for-루프가 수행되는 횟수이다. 이 횟수는 T아질수록 커지도록 구성되어있다.  
그리고 이웃해인 neighbor가 더 우수한 경우라고 결정이 되면 newSol이 neighbor이 되어서 진행되고 또 currentSol.CVRMSE - newSol.CVRMSE의 차이는 d가 된다. 이때의 d는 유전 알고리즘의 적합도와 비슷하다고 말 할 수있다. 이러한 방식으로 계속 돌아가면서 마지막에는 초기T를 일정 비율 알파로 감소시킨다. 이것이 모의 담금질의 핵심이다. 초기 T 즉 온도가 계속 냉각율인 알파에 따라서 줄어들면서 점점 더 우수한 해를 찾을 수 있는 것이다. 위에서 말한 것처럼 온도가 클때에는 자유도가 높아져서 해의 값이 정확하지 않지만 점점더 냉각율을 곱해 가면서 이 온도가 낮아지면서 해에 가까운 수를 출력해 낼 수 있다.  
## 모의 담금질의 이웃해 설정  
모의 담금질에서 가장 중요한 것은 이웃해 설정이라고 생각한다. 이 이웃해를 어떻게 설정하느냐에 따라서 아마도 모의 담금질의 효율성이 높아지거나 낮아질 것이다. 책에는 이웃해를 정의하는 방법은 3가지라고 나와있다. 삽입,교환,반전이다. 그러나 이 책에서는 경로찾기 문제이기 때문에 내가 구하려고 하는 그래프의 값과는 다르다. 아마도 그래프의 이웃해를 잘 설정하려면 위에서 내가 구했던 식이 1차식의 형태인 y=ax+b의 형태를 띄기 때문에 a와 b의 값을 서로 변환시키거나 아니면 b와 a의 순서를 바꾸어서 이웃해를 설정하여 푸는것도 나쁘지 않은 선택이라고 생각한다.
