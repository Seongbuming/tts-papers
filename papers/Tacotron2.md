# Tacotron 2

## 요약

<table>
    <tr>
        <th>DOI</th>
        <td><a href="https://doi.org/10.48550/arXiv.1712.05884">https://doi.org/10.48550/arXiv.1712.05884</a></td>
    </tr>
    <tr>
        <th>원제</th>
        <td>NATURAL TTS SYNTHESIS BY CONDITIONING WAVENET ON MEL SPECTROGRAM PREDICTIONS</td>
    </tr>
    <tr>
        <th>연도</th>
        <td>2018</td>
    </tr>
    <tr>
        <th>학술지 구분</th>
        <td>SCI</td>
    </tr>
    <tr>
        <th>학술지</th>
        <td>IEEE-ACM TRANSACTIONS ON AUDIO SPEECH AND LANGUAGE PROCESSING</td>
    </tr>
    <tr>
        <th>대상언어</th>
        <td>영어</td>
    </tr>
    <tr>
        <th>종단점</th>
        <td>Text → Spectrogram → Waveform</td>
    </tr>
    <tr>
        <th>데이터 타입</th>
        <td>&lt;text, audio&gt;</td>
    </tr>
    <tr>
        <th>데이터 분량</th>
        <td>24시간 30분</td>
    </tr>
    <tr>
        <th>보코더</th>
        <td>WaveNet</td>
    </tr>
    <tr>
        <th>평가지표</th>
        <td>MOS</td>
    </tr>
</table>

## 내용

### Previous works

- Deep Voice 3: 이것과 유사한 접근 방식
- Char2Wav: neural vocoder를 사용하는 E2E TTS에 대해 또 다른 유사한 접근 방식
- 상기 연구들은 different intermediate representations (traditional vocoder features)를 사용하고 모델 구조는 크게 다름

### Contributions

- ~~linguistic, duration, F0 feature 대신 WaveNet의 input 조절로 mel spectrograms를 사용하는 것이 미치는 영향 평가~~
- 언어 및 음향 Feature의 도메인 지식을 필요로 하던 WaveNet의 입력값을 훈련된 신경망으로 대체하여 speech synthesis pipeline 단순화함(in [Tacotron](./Tacotron.md))
- compact acoustic intermediate representation을 사용하여 WaveNet 아키텍처의 크기를줄임

### Limitations & Future works

- 평가자의 의견에 따르면, 이름 등을 처리할 때 발음에 어려움을 겪는 경우가 있음
- MOS 부정 응답의 주된 이유는 가끔 잘못된 발음

### Architecture

![Fig 1. Tacotron 2 시스템 아키텍처의 블록 다이어그램](../images/Tacotron2%201.png)
Fig 1. Tacotron 2 시스템 아키텍처의 블록 다이어그램

- 텍스트에서 직접 음성 합성하기 위한 신경망 아키텍처
- 문자 임베딩을 Mel-scale spectrograms에 매핑하는 Recurrent sequence-to-sequence feature 예측 네트워크로 구성되며, 이어서 ‘수정된 WaveNet’ 모델이 spectrograms에서 시간 영역 waveforms를 합성하는 vocoder 역할을 함