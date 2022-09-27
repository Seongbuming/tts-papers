# Tacotron

## 요약

<table>
    <tr>
        <th>DOI</th>
        <td><a href="https://doi.org/10.48550/arXiv.1703.10135">https://doi.org/10.48550/arXiv.1703.10135</a></td>
    </tr>
    <tr>
        <th>원제</th>
        <td>TACOTRON: TOWARDS END-TO-END SPEECH SYNTHESIS</td>
    </tr>
    <tr>
        <th>연도</th>
        <td>2017</td>
    </tr>
    <tr>
        <th>학술지 구분</th>
        <td>SCI</td>
    </tr>
    <tr>
        <th>학술지</th>
        <td>Interspeech</td>
    </tr>
    <tr>
        <th>언어</th>
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
        <td>Griffin-Lim</td>
    </tr>
    <tr>
        <th>평가지표</th>
        <td>MOS</td>
    </tr>
</table>

## 내용

### Previous works

기존 TTS 모델은

- 복잡한 파이프라인
- 통계학적 Parametric TTS
    1. 언어학적 특성 추출하는 Text frontend
    2. 음성 길이 예측 모델
    3. 음성 특성 예측 모델
    4. 복잡한 Signal processing 기반 Vocoder
- 전문적인 지식을 기반으로 하며 디자인하기 어려움
- 개별 학습되므로 각 컴포넌트에서 발생한 에러가 축적됨

### Contributions

- Character sequence로부터 직접 음성을 합성하는 E2E TTS 모델
    - 라벨링된 <text, audio> pair 데이터만으로 학습할 수 있음(E2E)
- 손이 많이 가는 언어학적 특성 추출, HNN aligner와 같은 복잡한 컴포넌트 필요하지 않음
- 언어 및 음향 feature의 생성을 데이터만으로 훈련된 단일 신경망으로 대체하여 전통적인 speech synthesis pipeline을 단순화

### Limitations & Future works

- 미래에는 text normalization이 불필요할 수도 있음
    - Tacotron은 simple text normalization을 사용함
- 모델을 많은 측면에서 더 조사해야 함
- Output layer, attention module, loss function, Griffin-Lim based waveform syntehsizer는 모두 개선 가능성이 있음
- 예를 들어 Griffin-Lim 출력은 audible artifacts를 가진다고 널리 알려져 있음
- 빠르고 높은 퀄리티의 neural-network-based spectrogram inversion을 연구하고 있음

### Architecture

![Fig 1. Tacotron 모델은 입력 문자열의 Raw spectrogram을 출력, 그것을 Griffin-Lim reconstruction algorithm에 제공하여 음성을 합성함](../images/Tacotron%201.png)

Fig 1. Tacotron 모델은 입력 문자열의 Raw spectrogram을 출력, 그것을 Griffin-Lim reconstruction algorithm에 제공하여 음성을 합성함

- Backbone: attention 기반 seq2seq 모델
- (Input) Characters → (Output) Raw spectrogram
- Fig 1: Encoder → Attention-based decoder → Post-processing net.의 모델 구조

#### 1. CBHG Module

![Fig 2. CBHG 모듈 (1-D convolution bank + highway network + bidirectional GRU), Lee at al. (2016)에 적용됨](../images/Tacotron%202.png)

Fig 2. CBHG 모듈 (1-D convolution bank + highway network + bidirectional GRU), Lee at al. (2016)에 적용됨

- CBHG에서 표현(representations)을 추출하는 모듈
- Input sequence → K sets의 1-D convolutional filters
    - k = 1, 2, …, K
    - k번째 set은 k너비(width)를 가지는 Ck(convolutional filter)
- 이러한 필터는 로컬 및 contextual information(unigrams, bigrams, up to K-grams 모델링과 유사)를 명시적으로 모델링
- convolution 출력이 함께 쌓고, local invariances를 증가시키기 위해 시간으로 max pooling 함.
- 이 때 stride를 1로 사용하여 원래 time resolution을 보존함
- 이후 fixed-width 1-D convolutions를 통과시킴
    - Conv1D projections in Fig 2.
- 다시 residual connection을 이용해 통과된 출력을 original input sequence와 더함
- Batch normalization은 매 convolution layer에서 사용됨
- convolution의 출력은 high level 특성을 추출하기 위해 multi-layer highway network를 통과함
    - Highway layers in Fig 2.
- 마지막으로, forward context와 backward context에서 sequential features를 추출하기 위해 Bidirectional GRU RNN을 사용함
- CBHG는 machine translation에서 영감 받음
    - 차이점은 non-casual convolutions, batch normalization, residual connections, stride=1 Max pooling
    - 이러한 modification으로 일반화(generalization) 효과 상승

#### 2. Encoder

- Encoder의 목표는 Robust sequential representations를 추출
- (Input) Character sequence
    - 각각의 character는 one-hot vector로 표현되고 continuous vector로 임베딩됨
- 그런 다음 pre-net이라 불리는 non-linear transformations 집합을 각 임베딩에 적용함
- 이 작업에서 dropout과 함께 bottleneck layer를 pre-net으로 사용하여 수렴을 돕고 일반화 개선
- CBHG 모듈은 pre-net outputs를 attention module에 사용되는 최종 encoder representation으로 변환
- CBHG기반 encoder가 overfitting을 줄일 뿐만 아니라 standard multi-layer RNN encoder보다 덜 잘못 발음함을 발견

![Table 1. 하이퍼파라미터와 Network architectures.<br>"conv-k-c-RELU"는 ReLu activation과 함께 width k와 c output channels가 있는 1-D convolution 의미함](../images/Tacotron%203.png)

Table 1. 하이퍼파라미터와 Network architectures.
”conv-k-c-RELU”는 ReLu activation과 함께 width k와 c output channels가 있는 1-D convolution 의미함

#### 3. Decoder

- 디코더로 content 기반 tanh attention decoder 사용
- stateful recurrent layer는 각 디코더 time step마다 attention query를 생성
- context vector와 attention RNN cell의 output은 concatenate돼서 decoder RNN의 입력으로 사용
- vertical residual connections와 함께 GRU stack을 사용함
- residual connection은 빨리 convergence될 수 있도록 도와줌
- 디코더의 타겟은 중요한 초이스
- Tacotron 모델은 직접 raw spectrogram을 예측하지만, 이는 speech signal과 text 사이에 alignment를 학습하기 위해서는 매우 불필요함
    - (이 작업은 실제로 seq2seq를 사용하는 동기)
- 이러한 불필요한 중복성(redundancy) 때문에, 모델에서 seq2seq 디코딩과 waveform 합성을 위해 다른 타겟을 사용함
- 수정되거나 훈련될 수 있는 inversion process에 대한 prosody(운율) 정보와 sufficient intelligibility를 제공하는 한, seq2seq 타겟은 매우 압축될 수 있음
- 80-band mel-scale spectrogram을 타겟으로 사용하지만 더 적은 bands나 cepstrum과 같은 더 간결한(concise) 타겟을 사용할 수 있음
- seq2seq 타겟에서 waveform으로 변환하기 위해서 post-processing network를 사용함

- decoder 타겟을 예측하기 위해 simple fully-connected output layer를 사용함
- 논문에서 발견한 중요한 트릭은 multiple, non-overlapping output frames를 각 decoder step마다 예측하는 것
- r frames를 한 번에 예측하는 것은 총 decoder steps를 r로 나누고 model size, 학습 시간, 추론 시간을 단축함
- 더욱 중요한 것은 이 트릭이 attention으로부터 학습되는 alignment가 더 빨리, 더 안정적으로 측정됨으로써 convergence speech가 증가했다는 것
- 이것은 인접한 speech frames들이 상관관계가 높고 각 character가 보통 여러 프레임으로 구성되기 때문
- 한 번에 하나의 프레임을 방출(emitting)하는 것은 모델이 여러 번의 단계에서 동일한 입력 토큰에만 집중하도록 강요함
- 여러 프레임을 방출(emitting)하는 것은 attention이 학습할 때 더 빨리 forward로 이동하게 만듬

- 첫 디코더 단계는 프레임을 나타내는 all-zero frame 상태에서 일어남
- 프레임으로 표현된 inference 상황에서 디코더 스텝 t에서, r개의 예측의 마지막 프레임은 스텝 t+1 디코더의 입력으로 사용됨
- 마지막 prediction을 입력으로 사용하는 것은 일반화할 수 없는 해결책(ad-hoc choice)임
    - 즉, 모든 r prediction들을 사용할 수도 있음
- 학습하는 동안은 항상 모든 r번째 ground truth frame을 디코더에 입력함
- 입력 프레임은 encoder에서 사용되었던 pre-net을 통과함
- scheduled sampling과 같은 테크닉을 사용하지 않기 때문에 pre-net에서 dropout은 모델을 일반화하는데 중요한 역할을 함
    - 그것은 출력 분포의 다양한 양상을 해결하기 위해 noise source를 공급함

#### 4. Post-processing net and Waveform synthesis

- post-processing net의 task는 seq2seq 타겟을 waveforms로 합성할 수 있는 타겟으로 변환하는 것
- 이 모델에서 Griffin-Lim을 합성기(synthesizer)로 사용하기 때문에, post-processing net은 linear-frequency scale로 샘플링된 스펙트럼 크기(spectral magnitude)를 예측하도록 학습함
- post-processing net의 다른 동기는 full decoder sequence를 볼 수 있다는 것
- 항상 왼쪽에서 오른쪽으로 실행되는 seq2seq와 대조적으로, 그것은 각 개별 프레임에 대한 예측 오류를 바로잡기 위해 forward, backward 정보를 모두 가짐
- 이 작업에서, 더 단순한 구조도 잘 작동할 수 있지만, post-processing net으로 CBHG 모듈을 사용함
- post-processing network의 개념은 매우 general함
- 이 모델은 vocoder parameters와 같은 alternative targets를 예측하거나 waveform 샘플을 직접 합성하는 WaveNet과 같은 neural vocoder로 사용할 수 있음
- 본 논문에서는 예측된 spectrogram의 waveform을 합성하기 위해 Griffin-Lim 알고리즘을 사용함
- Griffin-Lim 알고리즘에 넣기 전에 magnitudes를 1.2제곱하는 것은 artifacts를 감소시킴
    - (harmonic enhanement 효과 때문일 것)
- Griffin-Lim 알고리즘이 50회 iteration을 한 후에 합리적으로 빠르게 수렴하는 것을 관찰함
    - (실제로 약 30회 반복도 충분해 보임)
- Griffin-Lim을 TensorFlow로 구현하였음
- Griffin-Lim은 간단하고 이미 좋은 성능을 내지만 spectrogram to waveform inverter를 빠르고 고성능으로 학습할 수 있는 점에서 선택되었음

#### Model Detail

- Table 1의 하이퍼파라미터 및 network architectures
- Hann windowing, 50 ms frame length, 12.5 ms frame shift, 2045-point Fourier transform과 함께 log magnitude spectrogram을 사용함
- pre-emphasis (0.97)이 도움이 되는 것을 발견함
- 모든 실험에서 4kHz sampling rate를 사용함
    - 더 큰 r 값(e.g. r=5)이 더 잘 작동하지만,
    - 이 논문에서는 MOS 결과를 위해 r=2(output layer reduction factor)를 사용함
- Adam optimizer (Kngma & Ba, 2015)를 사용하고 learning rate decay는 각각 500K, 1M, 1M global steps 이후, 0.001에서 시작하여 0.0005, 0.0003, 0.0001로 줄임
- seq2seq decoder(mel-scale spectrogram)과 post-processing net(linear-scale spectrogram) 모두에 대해 simple l1 loss를 사용함
    - 두 loss는 같은 가중치를 가짐
- 모든 sequence는 max length에 맞춰 패딩되고 32 batch size를 사용ㅇ함
- 패딩된 부분은 zero-padded frames로 마스킹해 사용됨
- 하지만 이런 방식으로 학습된 모델이 언제 emitting outputs를 멈추는지를 모르고 마지막에 반복된 소리를 만들어냄
- 이 문제를 해결하기 위한 간단한 방법은 zero-padded frames를 재구성하는 것

