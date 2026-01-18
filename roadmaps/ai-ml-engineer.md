# AI/ML Engineer Roadmap 2026

A step-by-step guide to becoming an AI/ML Engineer in 8 months.

```
                                      AI/ML ENGINEER ROADMAP
    ┌───────────────────────────────────────────────────────────────────────────────────────┐
    │                                                                                       │
    │  Month 1-2        Month 3-4        Month 5-6        Month 7          Month 8         │
    │  ────────         ────────         ────────         ────────         ────────        │
    │                                                                                       │
    │  ┌────────┐       ┌────────┐       ┌────────┐       ┌────────┐       ┌────────┐      │
    │  │ Python │──────>│  ML    │──────>│  Deep  │──────>│  LLMs  │──────>│  MLOps │      │
    │  │ & Math │       │Fundamentals    │Learning│       │& GenAI │       │& Deploy│      │
    │  └────────┘       └────────┘       └────────┘       └────────┘       └────────┘      │
    │      │                │                │                │                │           │
    │      v                v                v                v                v           │
    │  ┌────────┐       ┌────────┐       ┌────────┐       ┌────────┐       ┌────────┐      │
    │  │ NumPy  │       │Scikit- │       │PyTorch │       │Hugging │       │  AWS   │      │
    │  │ Pandas │       │ Learn  │       │  /TF   │       │  Face  │       │ Sagemaker     │
    │  └────────┘       └────────┘       └────────┘       └────────┘       └────────┘      │
    │                                                                                       │
    └───────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Month 1-2: Python & Mathematics

### Week 1-2: Python for Data Science

**Learn:**
- Python fundamentals
- NumPy arrays and operations
- Pandas DataFrames
- Data manipulation and cleaning
- Matplotlib and Seaborn visualization

**Practice:**
```python
# Master these operations
import numpy as np
import pandas as pd

# Array operations
arr = np.array([1, 2, 3])
arr.reshape(), arr.dot(), np.linalg.inv()

# DataFrame operations
df.groupby(), df.merge(), df.pivot_table()
df.apply(), df.fillna(), df.drop_duplicates()
```

**Interview Prep:**
- [Python Interview Questions](../interview-questions/programming/python.md)

### Week 3-4: Mathematics for ML

**Learn:**
- Linear algebra (vectors, matrices, eigenvalues)
- Calculus (derivatives, gradients, chain rule)
- Probability (Bayes theorem, distributions)
- Statistics (mean, variance, hypothesis testing)

**Key Concepts:**
```
Linear Algebra:
- Matrix multiplication
- Dot product
- Eigendecomposition
- SVD (Singular Value Decomposition)

Calculus:
- Partial derivatives
- Gradient descent
- Chain rule for backpropagation

Probability:
- Conditional probability
- Bayes' theorem
- Normal, Bernoulli, Poisson distributions
```

---

## Month 3-4: Machine Learning Fundamentals

### Week 1-2: Supervised Learning

**Learn:**
- Linear Regression
- Logistic Regression
- Decision Trees and Random Forests
- Support Vector Machines
- Model evaluation metrics

**Practice:**
```python
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, f1_score

# Build models for:
- House price prediction (regression)
- Customer churn prediction (classification)
- Credit risk scoring
```

### Week 3-4: Unsupervised Learning & Feature Engineering

**Learn:**
- K-Means clustering
- PCA (dimensionality reduction)
- Feature scaling and encoding
- Handling missing values
- Feature selection techniques

**Practice:**
```python
from sklearn.cluster import KMeans
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler

# Projects:
- Customer segmentation
- Anomaly detection
- Recommendation system basics
```

---

## Month 5-6: Deep Learning

### Week 1-2: Neural Network Fundamentals

**Learn:**
- Perceptrons and activation functions
- Backpropagation
- Optimization (SGD, Adam)
- Regularization (dropout, batch norm)
- PyTorch or TensorFlow basics

**Practice:**
```python
import torch
import torch.nn as nn

class NeuralNetwork(nn.Module):
    def __init__(self):
        super().__init__()
        self.linear1 = nn.Linear(784, 256)
        self.linear2 = nn.Linear(256, 10)
        
    def forward(self, x):
        x = torch.relu(self.linear1(x))
        return self.linear2(x)
```

**Interview Prep:**
- [PyTorch Interview Questions](../interview-questions/ai-ml/pytorch.md)
- [TensorFlow Interview Questions](../interview-questions/ai-ml/tensorflow.md)

### Week 3-4: CNNs and RNNs

**Learn:**
- Convolutional Neural Networks
- Image classification
- Transfer learning (ResNet, VGG)
- Recurrent Neural Networks
- LSTMs and sequence modeling

**Practice:**
```python
# Projects:
- Image classifier with transfer learning
- Sentiment analysis with RNNs
- Time series prediction
```

---

## Month 7: LLMs & Generative AI

### Week 1-2: Transformer Architecture

**Learn:**
- Attention mechanism
- Transformer architecture
- BERT, GPT models
- Tokenization
- Fine-tuning strategies

**Practice:**
```python
from transformers import AutoTokenizer, AutoModel

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
model = AutoModel.from_pretrained("bert-base-uncased")

# Projects:
- Text classification with BERT
- Named Entity Recognition
- Question Answering
```

**Interview Prep:**
- [Hugging Face Interview Questions](../interview-questions/ai-ml/hugging-face.md)

### Week 3-4: Generative AI Applications

**Learn:**
- Prompt engineering
- RAG (Retrieval Augmented Generation)
- Vector databases (Pinecone, Weaviate)
- LangChain and LlamaIndex
- Fine-tuning LLMs (LoRA, QLoRA)

**Practice:**
```python
# Build:
- RAG chatbot with document retrieval
- Code assistant with context
- Multi-modal application
```

---

## Month 8: MLOps & Production

### Week 1-2: Model Deployment

**Learn:**
- Model serialization (ONNX, TorchScript)
- REST APIs for ML (FastAPI, Flask)
- Containerization with Docker
- Model versioning (MLflow, DVC)
- A/B testing for models

**Practice:**
```python
# Deploy a model:
from fastapi import FastAPI
import torch

app = FastAPI()

@app.post("/predict")
def predict(data: InputData):
    tensor = preprocess(data)
    prediction = model(tensor)
    return {"prediction": prediction.item()}
```

### Week 3-4: Cloud ML Platforms

**Learn:**
- AWS SageMaker
- Google Vertex AI
- Azure ML
- Model monitoring
- Drift detection

**Practice:**
```python
# SageMaker deployment
import sagemaker
from sagemaker.pytorch import PyTorchModel

model = PyTorchModel(
    model_data='s3://bucket/model.tar.gz',
    role=role,
    framework_version='2.0'
)
predictor = model.deploy(instance_type='ml.m5.large')
```

---

## Skills Checklist

### Must Have

- [ ] Python (NumPy, Pandas, Matplotlib)
- [ ] Machine Learning fundamentals
- [ ] Deep Learning (PyTorch or TensorFlow)
- [ ] Data preprocessing and feature engineering
- [ ] Model evaluation and validation
- [ ] SQL for data extraction
- [ ] Git version control

### Good to Have

- [ ] LLMs and Transformers
- [ ] RAG and vector databases
- [ ] MLOps (MLflow, DVC)
- [ ] Cloud ML platforms (SageMaker, Vertex)
- [ ] Docker and Kubernetes basics
- [ ] Spark for big data

### Nice to Have

- [ ] Reinforcement learning
- [ ] Computer vision (advanced)
- [ ] NLP (advanced)
- [ ] Distributed training
- [ ] Custom model architectures

---

## Interview Preparation

### Technical Questions by Topic

| Topic | Questions | Link |
|-------|-----------|------|
| Python | 12 | [python.md](../interview-questions/programming/python.md) |
| TensorFlow | 12 | [tensorflow.md](../interview-questions/ai-ml/tensorflow.md) |
| PyTorch | 12 | [pytorch.md](../interview-questions/ai-ml/pytorch.md) |
| Hugging Face | 12 | [hugging-face.md](../interview-questions/ai-ml/hugging-face.md) |

### Common ML System Design Questions

1. Design a recommendation system for an e-commerce platform
2. Design a fraud detection system for real-time transactions
3. Design a content moderation system using LLMs
4. Design a search ranking system with ML
5. Design an ML pipeline for 10TB of daily data

---

## Project Ideas

### Beginner
- Titanic survival prediction
- MNIST digit classification
- Sentiment analysis on reviews

### Intermediate
- Customer churn prediction pipeline
- Image classification with transfer learning
- Text summarization with transformers

### Advanced
- RAG chatbot with custom knowledge base
- Real-time object detection system
- Recommendation system with embeddings

---

## Salary Expectations (2026)

| Level | US (Remote) | India (Metro) |
|-------|-------------|---------------|
| Junior ML Engineer | $90-130K | ₹10-20 LPA |
| ML Engineer | $140-180K | ₹20-40 LPA |
| Senior ML Engineer | $180-250K | ₹40-70 LPA |
| Staff ML Engineer | $250-350K | ₹70-1.2 Cr |

---

## Resources

### Books
- "Hands-On Machine Learning" by Aurélien Géron
- "Deep Learning" by Ian Goodfellow
- "Designing Machine Learning Systems" by Chip Huyen

### Courses
- fast.ai - Practical Deep Learning
- Stanford CS229 - Machine Learning
- Stanford CS231n - CNNs for Visual Recognition

### Practice
- Kaggle competitions
- Papers with Code
- Hugging Face tutorials

---

## Next Steps

1. **Master Python first** - It's the foundation for everything
2. **Build projects** - Kaggle competitions are great practice
3. **Read papers** - Stay current with research
4. **Contribute to open source** - Hugging Face, PyTorch
5. **Specialize** - NLP, Computer Vision, or MLOps

---

*Want to deploy ML models on real cloud infrastructure? [Try DeployU](https://deployu.ai?ref=github) - Build RAG applications, deploy models, and learn MLOps on real AWS/GCP.*
