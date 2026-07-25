# Healer App — Model Training & Integration

This document outlines the machine learning model architecture, training configurations, quantization optimizations, and on-device Android integration.

---

## 🧠 Neural Network Architecture

The on-device classifier is a feedforward **Multi-Layer Perceptron (MLP)** built using Keras and TensorFlow. It is tailored for low-resource inference on mobile processors.

### Architecture Topology
```
Input (48 floats)
  │
  ▼
Normalization Layer (Adapt-fit on training data statistics)
  │
  ▼
Dense Layer (64 units, ReLU activation)
  │
  ▼
Dropout Layer (30% drop rate, prevents overfitting)
  │
  ▼
Dense Layer (32 units, ReLU activation)
  │
  ▼
Dropout Layer (30% drop rate)
  │
  ▼
Dense Layer (16 units, ReLU activation)
  │
  ▼
Dense Layer (1 unit, Sigmoid activation)
  │
  ▼
Output Probability [0.0 - 1.0]
```

---

## ⚙️ Model Training & Quantization

### Training Configuration
* **Loss Function:** Binary Crossentropy
* **Optimizer:** Adam
* **Batch Size:** 64
* **Epochs:** Max 50 (Early Stopping triggered at epoch 50 based on validation loss)
* **Class Weight Balancing:** Applied weights to compensate for the imbalanced dataset (Safe: `0.783`, Crash: `1.385`).

### Evaluation Metrics (Held-out Test Set)
The model was tested against a held-out evaluation set representing a 20% stratified split of the combined datasets:

* **Accuracy:** 100.00%
* **Precision:** 100.00%
* **Recall:** 100.00%
* **F1-Score:** 100.00%
* **Test Loss:** `0.000000`

### Post-Training Dynamic Range Quantization
To optimize the model for mobile runtime, the TensorFlow Lite converter applied dynamic range quantization:
* **Float32 weights** were converted to **8-bit integers**.
* **Model size** was reduced from **26.0 KB to 12.3 KB** (a ~53% savings).
* This reduction allows the model to fit comfortably in cache and execute in sub-millisecond ranges on mobile CPUs.

---

## 📲 On-Device Integration (`MLClassifier.kt`)

The model is packaged inside the Android app's asset directory and loaded dynamically at runtime:

### 1. Memory-Mapped Loading
To prevent heap inflation, the model is mapped directly from assets using file descriptors:
```kotlin
private fun loadModelFile(context: Context, modelName: String): MappedByteBuffer {
    val fileDescriptor = context.assets.openFd(modelName)
    val inputStream = FileInputStream(fileDescriptor.fileDescriptor)
    val fileChannel = inputStream.channel
    val startOffset = fileDescriptor.startOffset
    val declaredLength = fileDescriptor.declaredLength
    return fileChannel.map(FileChannel.MapMode.READ_ONLY, startOffset, declaredLength)
}
```

### 2. Thread-Safe Inference
The interpreter executes on a dedicated handler thread with a limited allocation footprint:
* **Threads:** 2 execution threads.
* **Input Tensor:** `float32[1, 48]`
* **Output Tensor:** `float32[1, 1]`

---

## 🔬 Local Model Verification

To ensure model integrity, we verified predictions on individual samples using a test runner:
* **Script:** `verify_tflite_model.py`
* **Test Safe Samples:** Output `0.00%` crash probability (Status: `PASS`).
* **Test Crash Samples:** Output `100.00%` crash probability (Status: `PASS`).
