# [AI] RAG(Retrieval-Augmented Generation)

## 서론

대규모 언어 모델(LLM)의 등장은 인간만이 가능하다고 여겨졌던 많은 분야에서 인공지능의 잠재력을 보여주었습니다. 방대한 양의 데이터를 기반으로 학습한 LLM은 인간 수준의 텍스트를 생성하고, 다양한 질문에 답변을 제공하며, 나아가 창의성이 요구되는 예술 분야에서도 두각을 나타내고 있습니다.

그러나 LLM이 보편화된 지금, 이를 상용화하려는 많은 기업은 오랜 문제에 직면하고 있습니다. 그 문제는 바로 '할루시네이션'입니다. 인공지능 챗봇과의 대화에 큰 기대를 품었던 사용자들은 LLM이 제공하는 부정확하고 불완전한 답변에 실망하곤 합니다. 이는 할루시네이션 문제와 밀접한 관련이 있습니다. 할루시네이션은 단순히 사실이 아닌 내용을 만들어내는 문제를 넘어, LLM이 데이터의 최신성이나 출처의 투명성 확보와 같은 '정보 검색 방식의 질적 전환'에 필수적인 기능을 갖추지 못했음을 시사합니다. 아무리 그럴듯해 보여도 실제로는 도움이 되지 않는 답변으로는 기존의 정보 검색 패러다임을 변화시킬 수 없을 것입니다.

## RAG 란?

이러한 LLM의 한계를 극복하기 위해 여러 방법이 제안되고 있습니다. 그중 하나는 Fine tuning으로, 사전 학습 모델(pre-trained model)에 특정 도메인(예: 의료, 법률, 금융)의 데이터를 추가 학습시켜 모델을 최적화하는 방식입니다. 이를 통해 모델은 특정 분야에 대한 전문 지식을 습득하고 정확하고 전문적인 답변을 제공할 수 있습니다. 그러나 Fine tuning은 시간과 비용이 많이 소요되고, 모델의 범용성이 저하될 수 있습니다.

반면 RAG는 외부 지식 소스와 연계하여 모델의 범용성과 적응력을 유지하면서도 정확하고 신뢰할 수 있는 답변을 생성할 수 있습니다. 즉, RAG는 LLM의 한계를 극복하면서도 그 장점을 살릴 수 있는 접근 방식이라고 할 수 있습니다.

RAG는 다음과 같은 장점을 가지고 있습니다.

1. **Fine tuning에 비해 시간과 비용이 적게 소요됩니다.**
    - 외부 데이터베이스를 활용하기 때문에 별도의 학습 데이터를 준비할 필요가 없습니다.
2. **모델의 일반성을 유지할 수 있습니다.**
    - 특정 도메인에 국한되지 않고 다양한 분야에 대한 질문에 답변할 수 있습니다.
3. **답변의 근거를 제시할 수 있습니다.**
    - 답변과 함께 정보 출처를 제공하여 답변의 신뢰도를 높일 수 있습니다.
4. **할루시네이션 가능성을 줄일 수 있습니다.**
    - 외부 데이터를 기반으로 답변을 생성하기 때문에 모델 자체의 편향이나 오류를 줄일 수 있습니다.

**RAG 자세히 알아보기**

그렇다면 RAG에 대해서 조금 더 자세하게 알아볼까요? RAG의 작동 과정을 단계별로 더욱 자세히 살펴보면 다음과 같습니다.

1. **데이터 임베딩 및 벡터 DB 구축**
    - RAG의 첫 번째 단계는 자체 데이터를 임베딩 모델에 통합하는 것입니다.
    - 텍스트 데이터를 벡터 형식으로 변환하여 벡터 DB를 구축합니다.
    - 이렇게 벡터화된 정보가 풍부한 데이터베이스는 Retriever(문서 검색기) 부분에서 사용자의 쿼리와 관련된 정보를 찾는 데 활용됩니다.
2. **쿼리 벡터화 및 관련 정보 추출 (증강 단계)**
    - 사용자의 질문(쿼리)을 벡터화합니다.
    - 벡터 DB를 대상으로 다양한 검색 기법을 사용하여 소스 정보에서 가장 관련성이 높은 부분 또는 상위 K개의 항목을 추출합니다.
    - 추출된 관련 정보는 쿼리 텍스트와 함께 LLM에 제공됩니다.
3. **LLM을 통한 답변 생성**
    - LLM은 쿼리 텍스트와 추출된 관련 정보를 바탕으로 최종 답변을 생성합니다.
    - 이 과정에서 정확한 출처에 기반한 답변이 가능해집니다.

![camearacalibration1](/assets/img/RAG_0.png)

## RAG가 최선일까?

과연 RAG의 기술이 최선일까? 질문을 하였을 때 아래와 같은 생각을 할 수 있다.

1. 더 발전된 모델인 Fusion-in-Decoder(FiD)나 Atlas를 고려하기
2. 키워드 검색과 벡터 검색을 함께 사용하기
3. Vector DB 대신 Knowledge Graph를 사용하기

그래서 본인은 2번을 근거하여 하이브리드 검색에 대해 스터디 해보고, 또한 3번을 근거하여 지식 그래프에 대하여 스터디를 진행 해보려고 한다.

그 후 RAG를 최종적으로 구현 하는 것 까지 진행 해보려고 한다.

## 하이브리드 검색

키워드 기반 검색인 Lexical Search와 유사도 기반 검색인 Semantic Search는 각각 장단점이 있고 사용이 유리한 상황이 서로 다르다.

### 하이브리드 검색의 주요 장점

- **Lexical Search** : OpenSearch와 같은 전통적인 검색 엔진에서 주로 사용되는 방법으로, BM25와 같은 희소 벡터 알고리즘을 통해 키워드 기반 매칭을 진행한다. 특정 도메인 용어를 검색하기에 용이하지만 오타 및 동의어에 취약하다.
- **Semantic Search** : 키워드가 일치하지 않더라도 의미론적으로 유사한 검색 결과를 반환한다. 검색 결과는 임베딩 품질에 의존도가 높다.

**결합된 검색 결과**:

- 하이브리드 검색은 키워드 검색과 벡터 검색의 결과를 결합하여, 키워드 일치와 의미적 유사성을 모두 고려한 최적의 검색 결과를 제공합니다. 이를 통해 검색 정확도를 높이고, 더 풍부한 정보를 사용자에게 제공할 수 있습니다.

### 하이브리드 검색의 활용 사례

하이브리드 검색은 특히 정보의 양이 방대하고 다양한 형태의 데이터가 존재하는 환경에서 효과적입니다. 예를 들어:

- **학술 논문 검색**: 키워드 검색을 통해 특정 주제의 논문을 찾고, 벡터 검색을 통해 논문의 의미적 유사성을 기반으로 연관된 논문을 추가로 찾을 수 있습니다.
- **전자상거래 사이트**: 제품 검색 시 사용자의 키워드 입력을 기반으로 제품을 찾고, 벡터 검색을 통해 사용자가 관심을 가질 만한 유사 제품을 추천할 수 있습니다.
- **대규모 데이터베이스 탐색**: 기업의 데이터베이스에서 특정 정보를 찾을 때 키워드 검색과 의미적 유사성을 결합하여 더 정확한 결과를 얻을 수 있습니다.

**LangChain을 통한 Hybrid Search 구현 방법**

```python
from langchain.retrievers import EnsembleRetriever
from langchain_community.vectorstores import OpenSearchVectorSearch

vector_db = OpenSearchVectorSearch(
            index_name=index_name,
            opensearch_url=oss_endpoint,
            embedding_function=embeddings,
            http_auth=http_auth, 
            is_aoss =False,
            engine="faiss",
            space_type="l2"
        )

# Sementic Retriever(유사도 검색, 3개의 결과값 반환)
opensearch_semantic_retriever = vector_db.as_retriever(
    search_type="similarity",
    search_kwargs={
        "k": 3
    }
)

search_semantic_result = opensearch_semantic_retriever.get_relevant_documents(query)

opensearch_lexical_retriever = OpenSearchLexicalSearchRetriever(
    os_client=os_client,
    index_name=index_name
)

# Lexical Retriever(키워드 검색, 3개의 결과값 반환)
opensearch_lexical_retriever.update_search_params(
    k=3,
    minimum_should_match=0
)

search_keyword_result = opensearch_lexical_retriever.get_relevant_documents(query)

# Ensemble Retriever(앙상블 검색)
ensemble_retriever = EnsembleRetriever(
    retrievers=[opensearch_lexical_retriever, opensearch_semantic_retriever],
    weights=[0.50, 0.50]
)

search_hybrid_result = ensemble_retriever.get_relevant_documents(query)
```

## Knowledge Graph

Vector DB의 한계를 보완해 줄 지식 그래프는 기존의 유사성 기준으로 검색했던 vector DB와 달리 키워드를 통해서 연결되어 있는 노드들을 검색하는 방법이다.

지식 그래프의 강점인 **관계성**과 **연결성**이 있는데 vector DB와 비교하며 관찰해보자

![camearacalibration1](/assets/img/RAG_1.png)

**관계성**

지식 그래프는 데이터 간의 직접적인 ‘관계성’으로 데이터를 매핑해 저장하기 때문에 고맥락이거나 포괄적인 질문에 대해 기존의 벡터 DB 방식보다 더욱 정확한 답변이 가능합니다.

예를 들어 ‘영업팀에 30대 직원들은 누가 있어?’와 같은 질문이나 ‘데일 카네기가 쓴 책을 모두 나열해줘’와 같은 포괄적인 질문에 대해 특정 관계로 연결된 데이터를 전부 가져올 수 있어 높은 정확성을 보이죠.

반면 기존의 벡터 DB는 데이터 청크를 한 번에 임베딩하기 때문에, 각 청크 내의 내용에 대한 고맥락 정보는 제외될 가능성이 높습니다. 즉 질문의 복잡도가 높을수록 벡터 DB에서는 원하는 정보를 정확히 반환하기가 어렵습니다.

**연결성**

![camearacalibration1](/assets/img/RAG_2.png)

지식 그래프는 ‘(노드) —(관계)→ (노드) —(관계)→ (노드) …’의 형태로 관련 있는 데이터를 모두 연결하여 저장하기 때문에 복잡한 질문에 대해서도 높은 답변 정확성을 보입니다.

예를 들어 ‘AI 윤리에 관해 OpenAI 창립자가 언급된 최신 뉴스를 알려줘’와 같은 질문에 대해 혼동 없이 정확한 답변을 도출할 수 있는데요.

(OpenAI) —(Is Founded)→ (창립자 리스트)의 관계에서 ‘창립자 리스트’ 노드를 가져와, ‘AI 윤리’라는 키워드와 함께 최신 뉴스를 검색하여 모든 공동 창립자에 대한 AI 윤리 관련 뉴스를 가져올 수 있죠.

반면 벡터 DB 기반 검색에서는 질문에 대한 유사도 검색이 데이터 청크마다 모두 독립적으로 수행되기 때문에 복잡한 질문에 대한 모든 맥락을 담지 못하고 특정 부분에 대한 답변만 가져오는 경우가 많습니다.

가령 위 질문에 대해 OpenAI 창립자 중 한 명에 대한 뉴스만 가져오거나, AI 윤리와 관련 없는 OpenAI 창립자만 언급된 뉴스를 가져오는 식입니다.
