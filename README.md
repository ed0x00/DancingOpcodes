# Dancing with Opcodes

**Android Malware Detection via Sonification and Deep Learning**

Master's thesis (1st level Master in Cyber Security), Università di Pisa, Department of Information Engineering, A.Y. 2024/2025
Candidate: Enrico Dolce · Lecturer: Dott. Giovanni Ciaramella

## Overview

This project detects Android malware with static analysis by "sonifying" each app: its opcode sequence is mapped to musical notes, turned into a spectrogram image, and classified by a convolutional neural network. The spectrogram keeps both the order of instructions and their frequency patterns, giving the model a compact fingerprint of the app's logic.

## Pipeline

```
APK → Smali (Apktool) → Opcode sequence → Note/frequency mapping → Spectrogram → CNN
```

1. **APK collection**: balanced set of benign and malicious apps.
2. **Smali disassembly**: decompile each APK with [Apktool](https://github.com/iBotPeaches/Apktool).
3. **Opcode extraction**: save the ordered opcode sequence as plain text.
4. **Frequency mapping**: group opcodes into functional families and map each family to one of 12 notes (C4 = 261.63 Hz … B4 = 493.88 Hz). Control flow sits on low notes; field access, object creation and method invocation (`invoke` → B4) on high notes.
5. **Spectrogram generation**: a Python script renders each sequence as a time-frequency image.
6. **Classification**: fine-tuned CNNs label each image as malware or trusted.

### Length normalization

Benign apps produce far longer opcode sequences than malware (≈232,553 vs ≈2,899 opcodes on average). To stop the model learning length instead of behavior, every sequence is fixed to ~2,500 opcodes (with randomized variance): longer ones are truncated, shorter ones are padded with the neutral note `D4`.

## Dataset

- 5,000 APKs: 2,500 malware (public malware repositories) and 2,500 trusted (legitimate repositories)
- Split 80/10/10: 4,000 train, 500 validation, 500 test

## Models and training

MobileNet and Inception, initialized with ImageNet weights and fine-tuned. Input size 100×100×3, batch size 32.

| Model | Learning rate | Epochs |
|---|---|---|
| Inception | 0.0001 | 30 |
| MobileNet | 0.0001 | 35 |

Higher learning rates (0.001, 0.0005) made training unstable for both architectures.

## Results (test set, 500 samples)

| Model | Loss | Accuracy | Precision | Recall | F1 | AUC |
|---|---|---|---|---|---|---|
| Inception | 0.377 | 0.948 | 0.948 | 0.948 | 0.948 | 0.955 |
| **MobileNet** | **0.250** | **0.958** | **0.958** | **0.958** | **0.958** | **0.983** |

MobileNet: 11 false negatives, 10 false positives. Inception: 11 false negatives, 15 false positives. MobileNet trained more stably and is the recommended model, especially for resource-constrained devices.

## Limitations

- A static model ages as new malware families and obfuscation techniques appear, so it needs periodic retraining.
- Static analysis is exposed to evasion: variants with the same behavior can have very different opcode sequences.
- Truncation drops information from the tail of long (mostly benign) sequences; sliding windows or other normalization schemes could avoid this.

## Future work

- Evaluate on obfuscated and packed samples to test robustness.
- Apply the same sonification approach to Windows PE and Linux ELF binaries.

## Related work

Builds on image-based (Ciaramella et al., 2025; Iadarola et al., 2021) and audio-based (Mercaldo & Santone, 2021; MalWave, Vasilellis et al., 2025) malware detection. Unlike those methods, it works on opcode sequences rather than raw bytes and feeds spectrograms straight into a CNN instead of relying on hand-crafted features.
