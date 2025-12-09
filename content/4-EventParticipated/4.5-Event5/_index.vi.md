---
title: "Sự kiện 5"
date: ""
weight: 5
chapter: false
pre: " <b> 4.5. </b> "
---
# **Báo Cáo Thu Hoạch Sự Kiện: “Building Agentic AI – Context Optimization with Amazon Bedrock”**

**Ngày tổ chức:** 05/12/2025  
 **Địa điểm:** Tầng 26, Bitexco Financial Tower

## **1\. Mục Tiêu Sự Kiện**

Sự kiện tập trung trình bày cách xây dựng **Agentic AI** (hệ thống AI đa tác vụ, đa agent) thông qua:

* Tối ưu ngữ cảnh (Context Optimization) khi xây dựng AI agent.

* Ứng dụng Amazon Bedrock cho multi-agent workflows.

* Giải pháp automation trong doanh nghiệp từ Diaflow và CloudThinker.

* Chia sẻ kinh nghiệm triển khai agent, thiết kế tool, xây dựng workflow


## **2\. Diễn Giả**

**Nguyen Gia Hung** — Head of Solutions Architect, AWS  
**Kien Nguyen** — Solutions Architect, AWS  
**Viet Pham** — Founder & CEO, Diaflow  
**Kha Van** — Community Leader, AWS  
**Thang Ton** — Co-Founder
**Henry Bui** — Head of Engineering

## **3\. Nội Dung Nổi Bật** 

### Session 1 – Agent Core (AWS)

* React Agent – thực thi tuần tự từng task.

* Thiết kế Tool trong Agent System.

* Tối ưu đánh giá và giám sát từ ngày đầu (Eval & Observability).

* Multi-agent & Multi-session: mỗi tác vụ/phòng ban vận hành theo một luồng agent riêng, trao đổi qua supervisor agent.

### Session 2 – Diaflow

**Vấn đề doanh nghiệp**

Theo trình bày từ Diaflow:

* **90% doanh nghiệp lãng phí \~20 giờ/tuần cho tác vụ lặp lại.**

* Nỗi lo chính: **AI có thể làm rò rỉ dữ liệu** khi xử lý.

**Giải pháp Diaflow**

Diaflow giải quyết bằng mô hình AI chạy hoàn toàn trên hạ tầng của doanh nghiệp:

* **Doanh nghiệp kiểm soát hoàn toàn luồng dữ liệu** thông qua MCP

* Hỗ trợ tích hợp nhiều dịch vụ Google & AWS.

* Ứng dụng Automation nội bộ:

  * Database Automation

  * Xử lý tài liệu

  * Knowledge base

  * Xây dựng internal tools/apps

**Lợi ích**

* **Process Efficiency**: Tự động hóa được đến 80% tác vụ lặp lại.

* **Cost-saving**: Triển khai trong vài phút, thay thế nhiều công cụ rời rạc.

* **Usability**: Giao diện đơn giản, ai cũng có thể tạo workflow tự động.

Kết hợp với AWS Bedrock

* Kết hợp **đa mô hình (multi-model)**.

* Tối ưu bảo mật dữ liệu ở mức doanh nghiệp.

* Khả năng tối ưu chi phí theo quy mô.

### Session 3 – CloudThinker

**Các thách thức Cloud doanh nghiệp**

* Chi phí hạ tầng dễ tăng vọt (Cost Explosion).  
* Hệ thống Cloud phức tạp, khó giám sát.  
* Khó phản ứng nhanh khi có sự cố.

**Giải pháp CloudThinker**

Một nền tảng “tất cả trong một” bao gồm:

* **Code Review Automation**

* **Incident Response Agent**

* **Operations & Optimization**

* **Security & Compliance Automation**

Giải pháp dựa trên multi-agent framework giúp:

* Tự động hóa vận hành

* Giảm chi phí

* Tối ưu hệ thống liên tục

* Nâng cao khả năng phản ứng sự cố

**Context Optimization**

Được nhấn mạnh là yếu tố quyết định hiệu suất của hệ thống agent:

* Input dành cho chatbot chỉ \~10% so với input cần cho agent system.

* Quá tải context gây giảm performance.

**Kỹ thuật tối ưu context:**

* Prompt Caching

* Context Compaction

* Tool Consolidation (gom tool vào MCI)

* Parallel Tool Calling

* Cross-Region Inference

Multi-agent Architecture

* Có **Supervisor Agent** điều phối.

* Các **Specialist Agents** xử lý tác vụ chuyên sâu.

* Kiến trúc Delegation giảm độ trễ & chia nhỏ workload.

Thách thức & Kinh nghiệm triển khai

* Khó nhất là **Agent Starter** – xây agent đầu tiên và mô hình tool calling.

* Cần thiết kế tool chuẩn ngay từ đầu.

* Bắt buộc phải có hệ thống **Eval & Observability** để tránh “AI out of control”.

## **4\. Bài học**

Kiến thức từ workshop hữu ích với các dự án AI/Cloud hiện tại:

* Xây dựng **multi-agent system** cho chatbot AI có thể phân tích, lập kế hoạch, và gọi API phức tạp.

* Áp dụng **context optimization** để giảm chi phí và tăng hiệu năng inference.

* Ứng dụng Diaflow cho workflow automation nội bộ, đặc biệt trong trích xuất dữ liệu và RAG.

* Sử dụng tư duy CloudThinker về giám sát, vận hành và phản ứng sự cố trong hệ thống AI/ML.


## **5\. Đánh Giá Chung**

* Sự kiện được tổ chức chuyên nghiệp, nội dung hiện đại và sát nhu cầu doanh nghiệp.

* Các ví dụ từ Diaflow và CloudThinker giúp người tham dự hình dung rõ ràng cách triển khai agent trong thực tế.

* Phần trình bày của AWS giúp xây nền tảng agentic AI đúng kỹ thuật, tập trung vào luồng dữ liệu và thiết kế tool.

