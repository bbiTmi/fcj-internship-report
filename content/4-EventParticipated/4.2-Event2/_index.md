---
title: "Event 2"
date: ""
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Report: “AI/ML & Generative AI on AWS”

### **1. Event Objectives**
The event provides an overview of **AI/ML/GenAI capabilities on AWS**, including how to leverage Foundation Models, Prompt Engineering, RAG, and AWS prebuilt AI services to address real-world use cases. It also introduces **Amazon Bedrock AgentCore** — a new platform for building and operating AI Agents at production scale.

### **2. Speakers**

- **Lâm Tuấn Kiệt** – Sr DevOps Engineer, FPT Software  
- **Danh Hoàng Hiếu Nghị** – AI Engineer, Renova Cloud  
- **Đinh Lê Hoàng Anh** – Cloud Engineer Trainee, First Cloud AI Journey  
- **Văn Hoàng Kha** – Community Builder  

### **3. Key Highlights**

#### **3.1 Generative AI on Amazon Bedrock**

**Foundation Models (FMs):**  
AWS provides a managed library of large models from providers such as Anthropic, Meta, and OpenAI, enabling users to customize models without training from scratch.

**Prompt Engineering:**  
Common techniques introduced:

- *Zero-shot:* model receives only a task description  
- *Few-shot:* model learns through a few provided examples  
- *Chain-of-Thought:* model is guided to explain reasoning step-by-step  

**RAG – Retrieval Augmented Generation:**  
A technique that enhances GenAI accuracy by adding external knowledge:

- **R – Retrieval:** fetch relevant information  
- **A – Augmentation:** inject retrieved data into the prompt  
- **G – Generation:** model generates more grounded and accurate responses  

Applications: contextual chatbots, enterprise search, real-time summarization.

**Amazon Titan Embeddings:**  
Embedding model that converts text into vectors for similarity search and RAG workflows, supporting multilingual scenarios.

**AWS AI Services:**  
Rekognition, Translate, Transcribe, Textract, Comprehend, Polly, Kendra, Personalize, Lookout, and others reduce development time by offering prebuilt AI capabilities.

**Demo:**  
The **AMZPhoto** facial recognition app demonstrated practical AI integration into real products.

---

#### **3.2 Amazon Bedrock AgentCore – Building Large-Scale AI Agents**

A new framework supporting the development and operation of AI agents:

- Automated and scalable workflows  
- Long-term memory and session management  
- Detailed access control and identity management  
- Integrations with Browser Tool, Code Interpreter, Memory Store  
- Supports major frameworks: CrewAI, LangGraph, LlamaIndex, OpenAI Agents SDK  
- Enhanced observability, tracking, and testing  

---

### **4. Key Takeaways**

- **Bedrock is the GenAI hub:** centralized access to multiple foundation models  
- **Prompt + RAG increases accuracy:** contextual data is key to high‑quality output  
- **Embeddings enhance search intelligence:** Titan Embeddings optimize retrieval  
- **AI Services accelerate development:** no need to build models from scratch  
- **AgentCore simplifies GenAI operations:** reduces complexity in scaling, memory, and monitoring  

---

### **5. Practical Applications**

1. Apply RAG for context‑aware GenAI projects.  
2. Use AWS AI services to accelerate product development cycles.  
3. Improve model performance using learned Prompt Engineering techniques.  
4. Consider AgentCore for projects requiring scalable, production-ready AI agents.  
