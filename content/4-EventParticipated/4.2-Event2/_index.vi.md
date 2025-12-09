---
title: "Sự kiện 2"
date: ""
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---
# Báo cáo thu hoạch: “AI/ML & Generative AI trên AWS”

### **1\. Mục tiêu sự kiện**
Sự kiện nhằm cung cấp cái nhìn tổng quan về **các khả năng AI/ML/GenAI trên AWS**, cách tận dụng mô hình nền tảng (Foundation Models), kỹ thuật Prompt Engineering, RAG và các dịch vụ AI dựng sẵn của AWS để giải quyết các bài toán thực tế. Đồng thời, sự kiện giới thiệu **Amazon Bedrock AgentCore** — nền tảng mới hỗ trợ xây dựng và vận hành AI Agents ở quy mô sản xuất.

### **2\. Diễn giả**

*  **Lâm Tuấn Kiệt** – Sr DevOps Engineer, FPT Software 

*  **Danh Hoàng Hiếu Nghị** – AI Engineer, Renova Cloud 

*  **Đinh Lê Hoàng Anh** – Cloud Engineer Trainee, First Cloud AI Journey 

*  **Văn Hoàng Kha** – Community Builder 

### **3\. Nội dung nổi bật**

#### **3.1 Generative AI trên Amazon Bedrock**

 **Foundation Models (FMs):**  AWS cung cấp thư viện mô hình lớn từ các nhà cung cấp như Anthropic, Meta, OpenAI… giúp người dùng nhanh chóng tùy chỉnh mô hình mà không cần tự huấn luyện.

 **Prompt Engineering:**  Các kỹ thuật phổ biến được trình bày:

*  *Zero-shot:* mô hình chỉ nhận mô tả nhiệm vụ. 

*  *Few-shot:* cung cấp một số ví dụ để mô hình học. 

*  *Chain-of-Thought:* yêu cầu mô hình trình bày suy luận từng bước. 

 **RAG – Retrieval Augmented Generation:**  Kỹ thuật cải thiện độ chính xác của GenAI bằng cách bổ sung thông tin phù hợp:

*  **R – Retrieval:** tìm kiếm dữ liệu liên quan. 

*  **A – Augmentation:** đưa dữ liệu này vào ngữ cảnh của prompt. 

*  **G – Generation:** mô hình tạo câu trả lời có căn cứ hơn.    Ứng dụng: chatbot theo ngữ cảnh, tìm kiếm doanh nghiệp, tóm tắt dữ liệu. 

 **Amazon Titan Embeddings:**  Mô hình embedding giúp chuyển văn bản thành vector phục vụ tìm kiếm tương đồng, hỗ trợ đa ngôn ngữ và tối ưu cho hệ thống RAG.

 **AWS AI Services – các dịch vụ AI dựng sẵn:**  Rekognition, Translate, Transcribe, Textract, Comprehend, Polly, Kendra, Personalize, Lookout… giúp rút ngắn thời gian phát triển sản phẩm AI.

 **Demo:**  Ứng dụng nhận diện khuôn mặt **AMZPhoto** minh họa cách tích hợp AI vào sản phẩm thực tế.

 ---

#### **3.2 Amazon Bedrock AgentCore – Xây dựng AI Agents ở quy mô lớn**

 Framework mới hỗ trợ phát triển và vận hành AI agent:

*  Tự động hóa và mở rộng workflow tác vụ 

*  Bộ nhớ dài hạn và quản lý phiên làm việc 

*  Quản lý quyền truy cập & danh tính 

*  Tích hợp với Browser Tool, Code Interpreter, Memory Store 

*  Hỗ trợ các framework phổ biến: CrewAI, LangGraph, LlamaIndex, OpenAI Agents SDK… 

*  Quan sát, giám sát và kiểm thử nâng cao 

 ---

### **4\. Bài học chính**

*  **Bedrock là trung tâm GenAI:** Dễ dàng truy cập nhiều mô hình lớn chỉ từ một dịch vụ. 

*  **Prompt \+ RAG giúp nâng độ chính xác:** Ngữ cảnh phù hợp quyết định chất lượng đầu ra. 

*  **Embeddings cải thiện khả năng tìm kiếm:** Titan Embeddings hỗ trợ truy xuất thông minh hơn. 

*  **AI Services giảm thời gian triển khai:** Không cần tự huấn luyện mô hình. 

*  **AgentCore đơn giản hóa vận hành GenAI:** Giảm độ phức tạp trong scaling, memory và monitoring. 

 ---

### **5\. Ứng dụng vào công việc**

1.  Áp dụng RAG cho các dự án GenAI yêu cầu truy xuất theo ngữ cảnh. 

2.  Sử dụng dịch vụ AI dựng sẵn để tăng tốc quá trình phát triển sản phẩm. 

3.  Tận dụng kiến thức về Prompt Engineering để cải thiện chất lượng mô hình. 

4.  Cân nhắc sử dụng AgentCore trong các dự án cần AI Agents hoạt động bền vững và có khả năng mở rộng.