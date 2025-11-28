# 🤖 치매 돌봄 로봇의 사용자 친밀도를 위한 AI Companion

<div align="center">
  <h2>🌿 WSL: <em>Wonderful Silver Life</em></h2>
  <p><strong>고령층의 삶에 정서적 연결을 제공하는 AI Companion</strong></p>
</div>

<br/>

<p align="center">
  <img src="./companion.png" alt="프로젝트 이미지" width="60%"/>
</p>

---
## 🔍 프로젝트 소개
### 프로젝트 개요 및 배경

고령화 사회의 심화로 인해 치매 환자와 같은 돌봄 대상자가 급증하고 있으며, 이에 따라 돌봄 인력 부족과 보호자의 부담이 사회적 문제로 대두되고 있습니다. 본 프로젝트는 낯선 음성과 영상으로 인한 노년층의 거부감을 줄이기 위해, 보호자의   얼굴과 목소리를 기반으로 한 맞춤형 AI 컴패니언을 개발하고자 시작되었습니다. 

### 활용 대상 및 목표 

치매 노인을 포함한 고령층이 사용하는 돌봄 로봇(다솜 K)에 적용되어, 보호자의 목소리와 얼굴로 말벗, 복약, 식사 등 안내 기능을 수행합니다. 음성과 영상 모두 보호자와 유사하게 생성함으로써 정서적 친밀감을 높이는 것이 핵심 목표입니다.

### 주요 기술 요소 요약

| 기술 | 설명 | 레포지터리 |
|------|------|------|
| **Zero-Shot Voice Cloning TTS** | 보호자의 예시 음성과 텍스트를 기반으로, 임의의 텍스트에 대해 유사한 음성을 생성 | [🔗 CosyVoice (TTS)](https://github.com/sogang-capzzang/CosyVoice) |
| **Rule-Based LipSync Video** | 보호자 사진과 음성을 이용해 입 모양이 자연스럽게 동기화된 영상을 생성 | [🔗 LipSync Module](https://github.com/sogang-capzzang/Real3DPortrait) |
| **SyncNet 기반 품질 평가** | 자동으로 LipSync 영상의 품질을 검증하는 품질 관리 시스템 | [🔗 Quality Control](https://github.com/sogang-capzzang/syncnet) |
| **Android Demo App** | TTS 및 LipSync 기능을 직접 시연 가능한 Android 애플리케이션 구현 | [🔗 Android Demo](https://github.com/sogang-capzzang/WSL-Application) |

## 🧠 전체 아키텍처

![파이프라인 아키텍처](./overall_architecture.png) 

본 시스템은 다솜K 로봇의 정서적 친숙함을 높이기 위해 두 가지 주요 모듈로 구성됩니다..
첫 번째는 Voice Cloning TTS 모듈로, 보호자의 reference 음성과 해당하는 문장, 그리고 전달할 문장에 해당하는 GPT Output(Gemini 등의 LLM 결과 텍스트)을 입력으로 받아 보호자의 목소리와 유사한 음성을 생성합니다. 생성된 음성은 실시간 TTS 말벗 기능을 통해 다솜K 로봇에 스트리밍됩니다.

두 번째는 LipSync 모듈로, Voice Cloning TTS 모듈에서 생성된 음성과 reference 보호자 사진을 입력받아 입 모양이 일치하는 Rule-Based LipSync Video를 생성합니다. 이 영상은 상황(식사, 복약 등)에 따라 다솜K 로봇에 재생됩니다. 생성된 영상은 SyncNet 기반의 품질 평가 모듈을 통해 음성과 입 모양의 동기화 여부를 검증할 수 있습니다.


## 👥 역할 분담

| 이름       | 역할 |
|------------|------|
| **김민준 (팀장)** | 프로젝트 총괄, TTS 최적화 및 Backend 구현, 보고서, 발표자료 작성 및 행정 업무 |
| **고운**         | Android application 개발, 보고서, 발표자료 작성, vast.ai 관리 |
| **이경연**       | TTS 개발 및 최적화, GitHub/WIKI/회의록 관리 |
| **김우영**       | Deep Fake Video (Lip Sync) 메인 개발, 논문 작성 및 포스터 제작 |
| **최호진**       | Deep Fake Video (Motion) 메인 개발, 논문 작성 및 포스터 제작 |
