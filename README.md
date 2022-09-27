# TTS Papers
TTS(Text-to-Speech) 논문 리뷰

| 별칭 | 원제 | 연도 | 학술지 구분 | 학술지 | 대상언어 | 종단점 | 데이터셋 | 데이터 구조 | 데이터 분량 | 언어적 특성 | 보코더 | 보코더 피처 | 평가지표 | 비고 |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| [Tacotron](./papers/Tacotron.md) | TACOTRON: TOWARDS END-TO-END SPEECH SYNTHESIS | 2017 | SCI | Interspeech | 영어 | Text → Spectrogram → Waveform | - | &lt;text, audio&gt; | 24시간 30분 | - | Griffin-Lim | - | MOS | - |
| [Tacotron2](./papers/Tacotron2.md) | NATURAL TTS SYNTHESIS BY CONDITIONING WAVENET ON MEL SPECTROGRAM PREDICTIONS | 2018 | SCIE | IEEE-ACM TRANSACTIONS ON AUDIO SPEECH AND LANGUAGE PROCESSING | 영어 | Text → Spectrogram → Waveform | - | &lt;text, audio&gt; | 24시간 30분 | - | WaveNet | - | MOS | - |
| WaveNet | WAVENET: A GENERATIVE MODEL FOR RAW AUDIO | 2016 | - | arxiv | 영어, 중국어(보통화) | Audo → Text → Audio | VCTK(English, Multi-speaker), built in Google’s North American English and Mandarin Chinese TTS System(English, Mandarin) | &lt;audio, audio&gt; (WaveNet), &lt;text, audio&gt;(Conditional WaveNet) | 44시간(VCTK, Multi-speaker), 24.6시간(영어) + 34.8시간(중국어) | - | WaveNet | log F0 | Paired Comparison, MOS | Multi-speaker Synthesis, Speech Recognition |
| Char2Wav | Char2Wav: End-to-End Speech Synthesis | 2017 | - | ICLR | 영어, 스페인어 | Text → Linguistic Features → Waveform | VCTK(English), DIMEX-100(Spanish) | &lt;text, audio&gt; | - | phone, syllable, word, phrase and utterance-level features | Griffin-Lim, WaveNet, WORLD | WORLD vocoder features Taylor (2009) 참고 | - | - |