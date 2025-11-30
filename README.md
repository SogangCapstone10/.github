LALM을 이용한 한/영 Audio Annotation 벤치마크 개발
고운1, 서동휘1, 이시욱1, 정진호1, 장두성♰
서강대학교 컴퓨터공학과1
서강대학교 인공지능학과 ♰
{rhdns156, xjkkm5918, mcv121, ggabri02, dschang}@sogang.ac.kr

Development of a Korean-English Audio Annotation Benchmark Using a Large Audio-Language Model
Woon Ko1, Donghwi Seo1, Siwook Lee1, Jinho Jeong1, Duseong Chang♰
Dept. of Computer Science and Engineering, Sogang University 1
Dept. of Artificial Intelligence, Sogang University ♰

요   약
오디오 인공지능 연구에서 대규모 주석 데이터셋은 필수적이나, Ground Truth(GT) 구축은 고비용의 수작업을 요구한다. 이러한 높은 비용 장벽은 영어 외 언어의 벤치마크 개발을 더욱 어렵게 만들며, 특히 한국어 벤치마크는 부재한 실정이다. 본 연구는 이러한 한계를 극복하기 위해, LALM(Large Audio-Language Model)과 LLM을 결합한 자동 GT 생성 파이프라인을 제안한다. 제안하는 파이프라인은 세 단계로 구성된다: (1) LALM(AF3)이 오디오 파일로부터 영어 캡션을 1차 생성하고, (2) 생성된 캡션과 원본 오디오 간의 교차 모달 유사도를 측정하여 의미론적으로 불일치하는 캡션을 검증 및 폐기하며, (3) 검증된 영어 캡션만을 LLM(GPT-4o)을 통해 고품질의 한국어 캡션으로 번역한다. 이 구조는 인간의 개입을 최소화하여 GT 구축의 비효율성 문제를 해결하는 동시에, 고품질의 한국어 GT를 자동으로 생성함으로써 한국어 벤치마크 부재 문제를 해결하고 한·영 병행 평가가 가능한 벤치마크를 구축하고자 한다.

 
1. 서  론
  오디오 인공지능 분야는 최근 대규모 멀티모달 모델의 발전과 함께 빠르게 확장되고 있으며, 모델의 성능을 평가하기 위한 벤치마크 구축은 핵심 과제이다. 그러나 그 구성 요소 중 하나인 GT(Ground Truth) 생성은 여전히 비효율적이고 고비용이며, 한국어 GT는 부재한 실정이다. 본 연구는 이러한 한계를 극복하기 위해 LALM(Large Audio-Language Model)을 이용해 GT를 구축 시 인간 개입을 최소화하고, 이를 기반으로 오디오 캡셔닝 벤치마크를 개발하는 것을 목표로 한다.
또한, 기존 벤치마크들이 모두 영어중심으로 구성된 한계를 극복하기 위해, 한국어 기반 GT를 함께 생성하고 한·영 병행 벤치마크를 구축하여 언어적 다양성을 확장한다. 이를 통해 GT 생성 과정에서 인간 의존도를 줄이고, 한국어 오디오-언어 연구의 확장 가능성을 제시하고자 한다.

2. 연구 배경
기존 오디오 캡셔닝 벤치마크 구축 방식에는 두 가지 주요 한계가 존재한다. 첫째, Ground Truth(GT) 생성의 비효율성이다. AudioCaps [1]나 Clotho [2]와 같은 인간 주석 기반 방식은 ________________________________________
※ 본 연구는 2025년 과학기술정보통신부 및 정보통신기획평가원의 "SW중심대학사업" (2024-0-00043, 기여율 50%)과 "멀티모달 AI 에이전트 시대에 적합한 실무형 AI인재 육성 프로그램"(RS-2025-25441313, 기여율 50%)의 지원을 받아 수행된 연구임.
고품질을 보장하나 막대한 시간과 비용이 소요된다. 이를 극복하기 위해 WavCaps [3]는 웹에서 수집한 공개 오디오 데이터의 설명문을GPT 기반 프롬프트로 변환하였으나 사람이 작성한 원문(description)에 의존하기 때문에 데이터 출처별 표현 일관성이 떨어지고 품질 편차가 발생한다는 문제가 존재한다.
  둘째, 한국어 오디오 벤치마크의 부재이다. AudioCaps, WavCaps 등 주요 벤치마크가 모두 영어 기반으로 구축되어, 한국어의 언어적 특성을 반영한 오디오-언어 모델의 학습 및 평가가 제한적인 실정이다.
본 연구는 이러한 한계를 극복하기 위해 ALM(Audio-Language Model)을 활용한 접근 방식을 제안한다. ALM은 기존 Language Model의 언어 이해 및 생성 능력을 오디오 도메인으로 확장한 모델로, 텍스트 입력에만 의존하는 LLM과 달리 오디오와 자연어를 함께 처리할 수 있다. 인간 작성 원문에 의존하지 않고 오디오 파일로부터 직접 캡션을 생성하여 GT 생성의 비효율성과 품질 편차 문제를 동시에 해결하고 일관성 있는 결과물을 확보하는 것을 목표로 한다.

3. 벤치마크
3.1 데이터
  본 연구에서 제안하는 벤치마크는 Clotho Dataset[2]의 오디오 파일을 기반으로 구축되었다. 본 벤치마크의 핵심 구성 요소인 영어 및 한국어 Ground Truth는 Clotho의 오디오 파일을 입력으로 하여 본 연구에서 제안하는 LALM 기반 GT 자동 생성 파이프라인을 통해 신규 생성하였다. 

3.2 Ground Truth 생성 파이프라인
3.2.1 파이프라인 구조
[그림 1] Ground Truth 생성 파이프라인
  본 연구에서는 [그림 1]와 같이 LALM 모델인 AF3를 이용한 3단계 Ground Truth(GT) 생성 파이프라인을 제안한다.
① 생성 (Generation): AF3가 '오디오 파일'과 '프롬프트'를 입력받아 영어 캡션을 1차 생성한다.
② 검증 (Validation): 생성된 캡션이 원본 오디오와 의미론적으로 일치하는지, 오디오-텍스트 인코더 간의 교차 모달(cross-modal) 유사도를 측정하여 검증한다. 임계값(τ)을 넘지 못하는 캡션은 폐기한다.
③ 번역 (Translation): 검증을 통과한 영어 캡션은 GPT-4o를 통해 고품질의 한국어 캡션으로 번역된다. 이 구조는 AF3의 생성 능력과 교차 모달 검증을 결합하여, 인간의 개입 없이도 일관성 있는 한/영 GT를 자동으로 구축할 수 있게 한다.

3.2.2 필터링
  생성된 캡션의 품질을 자동 검증하기 위해, MACE[5]에서 제안된 방식을 활용하여 오디오–텍스트 유사도 및 유창성 기반 필터링 모듈을 설계하였다. 오디오와 캡션의 임베딩은 MS-CLAP [6]을 사용하여 추출하며, 두 벡터 간 코사인 유사도(cosine similarity)를 통해 의미 정합도를 측정한다.  
오디오–텍스트 유사도 점수는 식 (1)과 같이 계산된다.

█(S_(audio-text)=cos(E_a (A),E_t (C))#(1) )

  여기서 E_a (A)는 오디오 임베딩, E_t (C)는 캡션의 텍스트 임베딩을 의미한다. 또한, FENSE의 방법을 적용하여 문장의 유창성 오류(Fluency Error)를 검출하고 이에 따라 패널티(FP)를 부여한다.  최종 점수는 아래 식 (2)와 같이 계산된다.

█(Score=S_(audio-text)×(1-α⋅FP)#(2) )

여기서 α는 가중치 계수이며, 임계값 이하의 점수를 가지는 캡션은 저품질로 판단되어 필터링되도록 설계하였다. 

3.3 평가 지표
  벤치마크에서는 생성된 캡션의 품질을 평가를 위해 다중 평가지표를 사용한다. 먼저 영어 캡션에 대해서는 단순한 어휘적 일치도 측정 방식이 아닌 의미론적 유사성을 평가하는 데 중점을 두었다. 이를 위해 오디오 캡셔닝 분야에서 널리 활용되는 Sentence-BERT, FENSE, SPIDEr-FL을 평가지표로 사용한다.
  한국어 캡션의 경우, 한국어의 특성을 반영하기 위해 한국어를 기반으로 사전학습된KoSimCSE와 KoBERT를 사용한다. KoSimCSE는 문장 수준의 의미 임베딩 유사도를, KoBERT는 단어 수준의 문맥적 일치도를 측정한다.
  또한, 한국어와 영어 캡션 모두에 공통적으로 적용 가능한 평가로 CLAIRA_A[7]를 도입하였다. CLAIRA_A는 LLM(Large Language Model) 기반의 평가 프레임워크이므로 한/영 모두에서 활용 가능하며, 점수 산출과 함께 자연어 설명(reasoning)을 제공한다. 본 연구에서는 이를 한·영 캡션의 공통 평가 프레임워크로 사용하였다.

3.4 번역 품질 평가 및 향상
  본 연구에서는 영어 원문 캡션을 기반으로 한국어 Ground Truth(GT)를 구축하기 위해 아래와 같은LLM, 번역 서비스대상으로 4가지 품질 지표를 활용하여 성능을 비교하였다.
표 1. 번역 모델 별 평가지표
Model
COMET-QE	BERT-QE	PPL 	Reverse-
BLEU
DeepL	-1.378	0.702	176.99	99.856
Papago	-1.435	0.701	208.53	35.403
GPT	-1.379	0.702	178.96	99.916
Gemini	-1.370	0.703	173.46	30.922
Claude	-1.396	0.700	233.20	35.688


[표 1]과 같이, GPT 4o 모델이 종합적으로 가장 높은 지표를 보여 최종 번역 모델로 선정하였다. 번역 프롬프팅 기법을 적용한 결과, COMET-QE 점수가 평균 0.24에서 0.32로 향상되었으며, 일정 점수 이하 샘플에 대해 재 번역을 수행한 경우 0.34, 재 번역 이후 다른 프롬프트를 이용하는 방식에서는 0.39까지 성능이 개선되었다.[표 2]
표 2. 번역 방식 별 평가지표
Method	COMET-QE 
Baseline	0.24
Few-shot	0.30
CoT	0.29
post-filtering	0.34
Prompt Switching	0.39

4. 실험
4.1 파이프라인 비교 실험
  본 연구는 LALM의 GT 생성 성능을 최적화하기 위해, 2000개의 오디오 샘플을 대상으로 세 가지 프롬프트 방식의 캡션 품질을 비교하였다. 비교 방식은 (1) keyword 미사용 (기본 프롬프트), (2) keyword 포함 (추출한 키워드를 프롬프트에 포함), (3) keyword 활용명령 포함 (키워드 활용 명령어를 프롬프트에 포함)이다. 각 방식이 생성한 캡션을 원본 Clotho GT와 CLAIRA_A 점수로 비교한 결과는 [표 3]과 같다.
표 3. CLAIRA_A를 이용한 Caption 생성 방식 별 점수
Captioning Method	Avg.	< 0.3 (%)	≥ 0.7 (%)
keyword 미사용 prompt	0.74	6.6	68.1
keyword 포함 prompt	0.61	23	55
keyword 활용명령포함 prompt	0.18	78	17
[표 3]의 결과, keyword 미사용 방식이 평균 0.74점으로 가장 높은 의미 정합도를 보였다. 또한 "좋은 캡션"(≥ 0.7) 비율이 68.1%로 가장 높았고, "나쁜 캡션"(< 0.3) 비율은 6.6%로 가장 낮았다.
  반면, 키워드를 프롬프트에 주입한 두 방식은 평균 점수가 하락하고 "나쁜 캡션" 비율이 상승하였다. 이는 키워드 정보의 직접적 개입이 오히려 LALM의 자연스러운 캡션 생성을 방해하여 의미 연관성을 저하시킬 수 있음을 시사한다.

4.2 필터링 모듈 성능 분석
  제안하는 MACE 기반 필터의 최적 임계값 선정을 위해 필터링의 정밀도(고품질 데이터 보존)와 재현율(저품질 데이터 제거)의 조화 평균인 F1-Score를 통해 임계값을 선정하였다. 성능 분석 결과0.42 에서 F1-Score가 최고점을 기록하여 이를 임계값으로 사용하였다.
표 4. MACE 기반 필터링 성능
CLAIRAA 점수	샘플 비율(%)	검출율(%)
< 0.3	6.6	43.2
≥ 0.7	68.1	6.2
		


  


분석 결과, CLAIRA_A점수 0.3 미만의 "나쁜 캡션” 중 과반을 넘긴43.2%를 성공적으로 검출(filter-out)하였다. 반면, 0.7 이상의 "좋은 캡션"에 대한 오검출률(False Positive Rate)은 6.2%로 나타났다. 이는 제안된 MACE 필터가 좋은 캡션은 대부분 보존하면서, 품질이 낮은 캡션과 오류는 효과적으로 선별함을 보여준다.

4.3벤치마크 이용한 평가
본 연구에서 제안하는 자동 생성 벤치마크(이하 Our GT)의 유효성을 검증하기 위해 WavCaps, Qwen, Whisper 모델을 대상으 로 비교 실험을 수행하였다. 기존 Clotho GT(Scenario A)와 Our GT(Scenario B)를 각각 정답으로 사용했을 때의 평가 양상을 분 석하였으며 결과는 [표 5]과 같다.
표 5: Clotho GT(Scenario A)와 제안하는 Our GT(Scenario B) 간
의 평가 결과 비교
 
	Scenario A	Scenario B	
	WavCaps	0.526	0.559	0.033	0.827
SBERT Sim	Qwen	0.407	0.450	0.043	0.877
	Whisper	0.392	0.412	0.020	0.881
	WavCaps	0.509	0.540	0.031	0.865
FENSE	Qwen	0.405	0.448	0.043	0.881
	Whisper	0.384	0.404	0.020	0.889
	WavCaps	0.519	0.507	0.013	0.801
CLAIRAA	Qwen	0.382	0.377	0.005	0.866
	Whisper	0.336	0.330	0.006	0.854
	WavCaps	0.270	0.347	0.078	0.546
SPIDEr-FL	Qwen	0.098	0.144	0.046	0.572
	Whisper	0.110	0.135	0.025	0.604
Metric	Model	Mean Score	Mean Diff  Correlation (r)

분석 결과, SBERT Sim, FENSE, CLAIRAA와 같은 의미론적 지표에서 0.8 이상의 높은 피어슨 상관계수(r)와 작은 평균 점수 차이(Mean Diff)를 보여, Our GT가 기존 데이터셋의 GT를 효과 적으로 대체할 수 있음을 확인하였다. SPIDEr-FL의 경우 상대 적으로 낮은 상관관계를 보였으나, 이는 두 시나리오 모두에서 해당 점수가 전반적으로 매우 낮게 형성되어 있어 작은 편차에 도 상관계수가 크게 민감하게 반응했기 때문이다. 두 시나리오 모두에서 모델 성능 순위(WavCaps > Qwen > Whisper)가 동일하게 유지되어 벤치마크로서의 변별력 또한 확인되었다.

5. 결  론
본 연구는 기존 오디오 벤치마크의 고비용 수작업 Ground Truth(GT) 생성 방식과 한국어 벤치마크의 부재 문제를 해결하고자 하였다. 이를 위해 LALM(AF3)의 캡션 생성, 교차 모달 검증, LLM(GPT-4o)의 기계 번역을 결합한 3단계 자동 GT 생성 파이프라인을 제안하고, 이를 기반으로 한·영 병행 벤치마크를 구축하였다.
실험 결과, 제안하는 파이프라인 중 ‘키워드 미사용 프롬프트’ 방식이 평균 0.74로 가장 우수한 캡션 생성 성능을 보였다. 또한, MACE 기반 검증 필터는 임계값 0.42에서 저품질 캡션을 43.2% 검출하고, 고품질 캡션에 대한 오검출률은 6.2%로 나타나 파이프라인의 신뢰성을 높였다. 프롬프팅 기법과 재 번역 전략을 적용하여 GPT 기반 한국어 캡션의 번역 품질을 체계적으로 평가하고 최적화하였으며, 원본과의 의미적 일관성을 확보하였음을 입증하였다. 

참고문헌
[1] C. D. Kim, B. Kim, H. Lee, and G. Kim, “AudioCaps: Gen-
erating captions for audios in the wild,” in Proceedings of the
2019 Conference of the North American Chapter of the As-
sociation for Computational Linguistics: Human Language
Technologies, Volume 1 (Long and Short Papers) (J. Burstein,
C. Doran, and T. Solorio, eds.), (Minneapolis, Minnesota),
pp. 119–132, Association for Computational Linguistics, June
2019.
[2] K. Drossos, S. Lipping, and T. Virtanen, “Clotho: an audio
captioning dataset,” in ICASSP 2020 - 2020 IEEE Interna-
tional Conference on Acoustics, Speech and Signal Process-
ing (ICASSP), pp. 736–740, 2020.
[3] X. Mei, C. Meng, H. Liu, Q. Kong, T. Ko, C. Zhao, M. D.
Plumbley, Y. Zou, and W. Wang, “Wavcaps: A chatgpt-
assisted weakly-labelled audio captioning dataset for audio-
language multimodal research,” IEEE/ACM Transactions on
Audio, Speech, and Language Processing, vol. 32, pp. 3339–
3354, 2024.
[4] A. Goel, S. Ghosh, J. Kim, S. Kumar, Z. Kong, S. gil Lee,
C.-H. H. Yang, R. Duraiswami, D. Manocha, R. Valle, and
B. Catanzaro, “Audio flamingo 3: Advancing audio intelli-
gence with fully open large audio language models,” 2025.
[5] S. Dixit, S. Deshmukh, and B. Raj, “Mace: Leveraging au-
dio for evaluating audio captioning systems,” in 2025 IEEE In-
ternational Conference on Acoustics, Speech, and Signal Pro-
cessing Workshops (ICASSPW), pp. 1–5, 2025 
[6] B. Elizalde, S. Deshmukh, and H. Wang, “Natural lan-
guage supervision for general-purpose audio representations,”
in ICASSP 2024 - 2024 IEEE International Conference on
Acoustics, Speech and Signal Processing (ICASSP), pp. 336–
340, 2024.
[7] T.-H. Wu, J. E. Gonzalez, T. Darrell, and D. M. Chan, “Clair-
a: Leveraging large language models to judge audio captions,”
2025.
