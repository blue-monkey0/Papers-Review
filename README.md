# TextRank: Bringing Order into Texts

| Writers | Journal/Conference | Year |
|---------|--------------------|------|
| **Rada Mihalcea and Paul Tarau** | EMNLP | 2004 |

본 논문은 텍스트 처리를 위한 그래프 기반 랭킹 모델인 **TextRank** 를 소개한다. 이 TextRank 모델은 자연어 applicaion에서 성공적으로 사용할 수 있다고 한다. <br>
특히, **키워드** 와 **문장 추출** 을 위한 두 가지 혁신적인 unsupervised 방법을 제안하는데, 이는 이전의 연구 결과와 비교했을 때 벤치마크에서 더욱 좋은 성능을 보여주었다고 한다. <br>
**Graph-based ranking algorithm** 은 지역 vertex별 특정 정보에만 의존하는 것이 아닌, 전체 그래프에서 recursive하게 계산된 전역 정보를 고려해서 그래프 내에서 vertex의 중요성을 결정하는 방법이다. <br>
반면, **Text-oriented Graph-based Ranking Methods** 은 Text Unit(단어, 어절, 문장) 간 연결 관계를 찾아 그래프로 재표현하고, 전체 text에서 도출된 knowledge를 반영하여 각 unit의 중요도를 산출 후, 주요 키워드/문장을 추출해준다.

TextRank의 작동원리로 텍스트를 그래프로 표현하기 위해 본 논문에선 **Keyword Extraction** (키워드 추출)과 **Sentence Extraction** (문장 추출) 이렇게 두 가지 task에 대해 방법론을 적용하였다. <br>
키워드 추출같은 경우 문서 내에 각 단어들이 vertex로 표현되고 각 단어들과 연결 관계에 따라서 edge가 생성이 된다. 그리고 최종적으로 각 단어들에 대해 중요도 점수를 산출해서 문서 내의 주요 키워드를 추출하게 된다. <br>
문장 추출같은 경우 vertex가 단어가 아닌 한 문장으로 이루어져있다는 차이점을 제외하곤 모두 동일한 알고리즘이다. 마찬가지로 각 문장마다 중요도 점수를 연결 관계를 통해서 산출하게 되고 이에 따라 주요 문장을 추출해서 문서 요약문으로 활용할 수 있다. <br>

 다음으로 키워드 추출을 위한 TextRank 방법론에 대해 살펴보겠다. <br>
 1. 기본 전처리
 - Tokenizing(토큰화)와 Part of Speech Tagging(Pos, 품사 태깅)으로 이루어진 기본 전처리 작업이 선행되어야 한다.
 - **Tokenization (토큰화)** 란, 주어진 corpus에서 token이라 불리는 단위로 나누는 작업을 뜻한다. 토큰의 단위가 상황에 따라 다르지만, 보통 의미있는 단위로 토큰을 정의한다. <br>
 입력이 *"Time is an illusion, Lunchtime double so!"* 라고 했을 때, 구두점을 제외시킨 토큰화 작업의 결과는 다음의 출력과 같다. *"Time", "is', "an", "illusion", "Lunchtime", "double", "so"* . 해당 토큰화는 뛰어쓰기를 기준으로 단순하게 하였으나, 보통 전치사, 관사를 제외시켜 토큰화를 하곤 한다. 그리고 선택적으로 불용어(stopwords) 집합을 미리 정의해둔 경우, 이 불용어 집합에 해당하는 단어들도 제외하게 된다.
 - **Part-of-speech tagging(품사 태깅)** 은 토큰화 과정에서 각 단어가 어떤 품사로 쓰였는지 구분해놓는 작업을 뜻한다. <br>
 입력이 *"Compatibility of systems of linear constraints over the set of natural numbers."* 라고 했을 때, 품사 태깅 작업의 결과는 다음의 출력과 같다. *"compatibility, systems, constraints, numbers"* : NN(noun, 명사), *"linear natural"* : JJ(adjective, 형용사).

2. Syntactic Filtering을 통한 Vertex 생성
- 해당 step에선 필터링을 통해 특정 품사만 남겨두고 나머지 토큰은 모두 제거해준다. 이 필터링은 그래프의 복잡도가 과도하게 증가하는 것을 방지하기 위해서 사용된다. 만약 모든 토큰들에 대해서 vertex를 생성하게 되면 그래프가 지나치게 커지기 때문에 그런 현상을 방지하고자 특정 품사나 혹은 특정 단어 집합에 속한 토큰들만 남겨두는 것이다.
- 본 논문은 필터링에 대해서 다양한 경우의 수로 실험을 진행했는데, 그 중 명사와 형용사 조합이 가장 최선의 결과를 도출했다고 말한다.
- 이 step까지 마치게 되면 각 토큰들은 다음의 그림과 같이 그래프의 vertex로 구성이 된다. <br>
- <img width=500 src=https://user-images.githubusercontent.com/48666867/149343642-d97460f1-4025-4446-b6ec-4a3a1cb284c3.png>

3. Co-occurrence relation에 따라 Edge 생성
- 해당 step에선 vertex간 연결 관계를 정의하게 된다. 연결 관계는 동시 등장 여부(Co-occurrence)에 따라 edge가 생성 되게 된다.
- **Co-occurrence** 는 window size N 이내에 동시 등장하는 두 vertex를 edge로 연결하는데 쓰인다. 본 논문에서는 예시로 N=2로 설정해주었다. 보통 N은 2에서 10까지 사용한다.
-  입력이 *"Compatibility of systems of linear constraints over the set of natural numbers."* 라고 했을 때, *linear* 를 기준으로 window size 2 이내에 동시 등장하는 단어는 *systems* 와 *constraints* 가 있다. 따라서 이들 vertex끼리 edge를 연결해주면 된다.
-  <img width=400 src=https://user-images.githubusercontent.com/48666867/149345753-da4427b7-4a0b-4dc3-9ba9-0d7e5af29070.png>

4. Importance Score 반복적으로 산출 (Iteration)
- 그래프가 Unweighted일 경우 다음 수식과 같다.
- <img width=400 src=https://user-images.githubusercontent.com/48666867/149350379-f61444d0-2934-4b10-9650-00668771564c.png>
- 여기서 d는 damping factor로 주로 0에서 1사이이다. 본 논문에서는 0.85값을 사용하였다. 그리고 초기 score는 1 또는 랜덤으로 지정해주었으며, 모든 vertex의 점수가 수렴할 때까지 반복해준다.
- 수렴 조건: 현재 Iteration의 중요도 점수 - 이전 Iteration의 중요도 점수 차이가 미리 지정한 threshold보다 작을 때, 반복 작업을 중단해준다.
- 그래프가 Weighted일 경우 다음 수식과 같다.
- <img width=400 src=https://user-images.githubusercontent.com/48666867/149351381-3fd2ced4-7b1c-4637-8dd4-755a6ff213d8.png>
- 4가지 그래프 조합에 대한 수렴 곡선을 보면 unweighted와 weighted 그래프와 비교했을 때 최종 수렴 score는 다르더라도 반복 횟수와 수렴 곡선은 거의 동일하다는 것을 알 수 있다.
- <img width=400 src=https://user-images.githubusercontent.com/48666867/149351782-e854e1ad-0ae4-435a-a59a-f29786680229.png>


