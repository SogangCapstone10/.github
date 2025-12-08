
# 🤖 LALM을 이용한 한/영 Audio Annotation 벤치마크 개발

<div align="center">
  <h2 K-E Audio Benchmark Project</h2>
  <p><strong>LALM과 LLM을 결합한 고품질 오디오 캡션 자동 생성 및 벤치마크 구축</strong></p>
</div>

<br/>

<p align="center">
  <img src="./project_banner.png" alt="프로젝트 대표 이미지" width="80%"/>
</p>

---

## 🔍 프로젝트 소개

### 📌 개요 및 배경
오디오 인공지능 연구에서 대규모 데이터셋은 필수적이지만, Ground Truth(GT)를 사람이 직접 작성하는 과정은 막대한 비용과 시간이 소요됩니다. 특히, 기존 벤치마크는 대부분 영어 중심으로 구성되어 있어 한국어 벤치마크는 전무한 실정입니다. 

본 프로젝트는 이러한 한계를 극복하기 위해 LALM(Large Audio-Language Model)과 LLM을 결합하여, 사람의 개입(후처리)을 최소화하면서도 고품질의 한/영 병행 오디오 캡션(GT)을 자동으로 생성하는 파이프라인을 개발하였습니다. 

나아가, CLAIR-A(LLM 기반 평가), MACE, KoBERT 등 한/영에 대한 다양한 평가지표를 도입하여, 구축된 데이터를 기반으로 AF3를 비롯한 타 ALM(Audio-Language Model)의 성능을 정량적으로 측정하고 분석할 수 있는 한/영 벤치마크를 완성하였습니다.

### 🎯 목표
1. **자동화된 GT 구축:** AF3(Audio Flamingo 3)와 GPT-4o를 연동하여 고비용의 수작업 라벨링 대체
2. **품질 검증 파이프라인:** MACE 및 CLAIR-A 등을 활용한 자동 필터링 및 평가 시스템 구축
3. **한국어 데이터 확보:** 영어 캡션을 기반으로 의미적 정합성이 확보된 한국어 GT 생성 및 벤치마크 공개

### 🛠️ 핵심 레포지토리 및 기술 스택

본 프로젝트는 각 기능별로 모듈화된 레포지토리를 운영하고 있습니다.

| 카테고리 | 기술/역할 | 레포지토리 |
|:---:|---|---|
| **Main Pipeline** | **GT 생성 및 모델 평가 프레임워크**<br>1. **Auto GT Pipeline**: AF3 생성 → MACE 필터링 → GPT 번역으로 이어지는 자동 GT 생성 모듈<br>2. **Evaluation**: 구축된 데이터셋(최소한의 후처리 적용)을 기반으로, 내장된 평가지표를 통해 모델 성능을 검증하는 평가 모듈 | [🔗 Auto-GT-Pipeline-Demo](https://github.com/SogangCapstone10/Auto-GT-Pipeline-Demo) |
| **System UI** | **웹 기반 시연 인터페이스**<br>사용자가 오디오 파일을 업로드하고 파이프라인의 전 과정(생성, 필터링, 번역)을 시각적으로 모니터링하며 테스트할 수 있는 프론트엔드 시스템 | [🔗 UI](https://github.com/SogangCapstone10/UI) |
| **LALM** | **Audio Flamingo 3 (AF3)**<br>오디오 입력을 받아 1차 영어 캡션을 생성하는 핵심 모델 | [🔗 audio-flamingo](https://github.com/SogangCapstone10/audio-flamingo) |
| **Eng Metric & Filter** | **AAC Metrics**<br>Sentence-BERT, SPIDEr-FL, FENSE 등의 영어 평가지표 모음.<br>그 중 **MACE**는 본 프로젝트의 필터링 모듈 핵심 알고리즘으로 응용 | [🔗 aac-metrics](https://github.com/SogangCapstone10/aac-metrics) |
| **Translation** | **번역 모듈**<br>필터링 이후의 영어 캡션에 대한 번역 모듈 | [🔗 translation](https://github.com/SogangCapstone10/translation) |
| **Common Metric** | **CLAIR-A**<br>LLM을 심판(Judge)으로 활용하여 한/영 캡션 모두에 적용 가능한 공통 평가 프레임워크 | [🔗 clair-a](https://github.com/SogangCapstone10/clair-a) |
| **Kor Metric** | **Korean Evaluation Metrics**<br>한국어 캡션 품질 평가를 위한 KoBERT(문맥) 및 KoSimCSE(의미 유사도) | [🔗 KoBERT](https://github.com/SogangCapstone10/KoBERT)<br>[🔗 KoSimCSE-SKT](https://github.com/SogangCapstone10/KoSimCSE-SKT) |


## 🧠 전체 아키텍처 (GT Generation Pipeline)

<p align="center">
  <img src="./pipeline_architecture.png" alt="파이프라인 아키텍처" width="90%"/>
</p>

본 시스템에서는 데이터셋의 Ground Truth에 대하여 **3단계 자동 생성 프로세스**를 따릅니다.

1. **Generation (AF3)**: `audio-flamingo`를 통해 오디오 파일로부터 1차 영어 캡션을 생성합니다. (Prompt Engineering 적용)
2. **Validation (Filtering)**: 생성된 캡션과 원본 오디오의 유사도를 `MACE` (in `aac-metrics`) 기반으로 측정하여, 의미적으로 불일치하는 캡션을 폐기합니다.
3. **Translation (LLM)**: 검증된 영어 캡션을 `GPT-4o`를 통해 한국어로 번역합니다. 이 과정에서 프롬프트 최적화를 통해 번역 품질을 극대화합니다.

최종적으로 구축된 데이터셋은 `CLAIR-A`, `KoBERT`, `SPIDEr-FL` 등 다각도의 평가지표를 통해 Large Audio-Langauage Model의 신뢰성을 검증할 수 있습니다.

## 👥 역할 분담

| 이름 | 역할 | 담당 업무 상세 |
|:---:|:---:|---|
| **정진호 (팀장)** | **Translation & Demo** | • 한국어 번역 프롬프트 설계 및 파이프라인 구축<br>• 웹 기반 시연 시스템 메인 개발 |
| **서동휘** | **PM & Presentation** | • 프로젝트 전체 일정 및 이슈 관리<br>• 논문 작성 총괄 및 투고 관리<br>• 최종 발표 자료 제작 및 발표 진행 |
| **고 운** | **Filtering & Evaluation** | • MACE 기반 오디오-텍스트 필터링 모듈 개발 및 임계값 실험<br>• 최종 벤치마크 모델 평가 실험 및 분석<br>• 최종 GT 생성 파이프라인 구축 |
| **이시욱** | **Prompting & Demo** | • AF3(LALM) 프롬프트 엔지니어링 및 최적화 실험<br>• 시연 시스템 기능 구현 및 연동 지원 |

## 📌 시연 영상
https://github.com/user-attachments/assets/ecbf9a3a-0cd4-43ae-bc16-3c51d1d924fa
---
<div align="center">
  <p>Department of Computer Science & Engineering, Sogang University</p>
</div>
