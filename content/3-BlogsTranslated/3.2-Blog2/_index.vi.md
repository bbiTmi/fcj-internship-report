---
title: "Blog 2"
date: ""
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---
# Cách thực thi AI model inference với GPUs trên Amazon EKS Auto Mode

Việc suy luận mô hình AI (AI model inference) bằng GPU đang trở thành thành phần cốt lõi trong các ứng dụng hiện đại, thúc đẩy các hệ thống gợi ý theo thời gian thực, trợ lý thông minh, tạo nội dung tự động, và các tính năng AI yêu cầu độ trễ thấp khác.Kubernetes đã trở thành nền tảng điều phối (orchestrator) được ưa chuộng để chạy các tác vụ suy luận, và các tổ chức muốn tận dụng những khả năng này trong khi vẫn tập trung hoàn toàn vào duy trì tốc độ đổi mới và thời gian ra mắt sản phẩm nhanh. Tuy nhiên thách thức đặt ra là: trong khi nhóm kỹ thuật nhận thấy giá trị của Kubernetes trong khả năng mở rộng linh hoạt và quản lý tài nguyên hiệu quả, họ thường bị chậm lại do phải học các Kubernetes concepts, quản lý cấu hình cụm (cluster configurations) và xử lý các bản cập nhật bảo mật. Điều này khiến họ phân tán khỏi mục tiêu chính: triển khai và tối ưu hóa mô hình AI. Đó chính là lý do [Amazon Elastic Kubernetes Service (Amazon EKS) Auto Mode](https://docs.aws.amazon.com/eks/latest/userguide/automode.html) ra đời. EKS Auto Mode tự động hóa việc tạo node, quản lý các [chức năng cốt lõi](https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html#addon-consider-auto), và xử lý nâng cấp và vá bảo mật, giúp bạn có thể chạy các tác vụ suy luận AI mà không phải gánh chịu chi phí vận hành phức tạp.

Trong bài viết này, chúng tôi sẽ hướng dẫn bạn triển khai nhanh chóng các tác vụ suy luận trên EKS Auto Mode. Chúng tôi cũng sẽ giới thiệu các tính năng quan trọng giúp đơn giản hóa việc quản lý GPU, trình bày các phương pháp triển khai mô hình tối ưu, và minh họa bằng một ví dụ thực tế thông qua việc triển khai các [mô hình mã nguồn mở của OpenAI](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-openai.html) thông qua [vLLM](https://docs.vllm.ai/en/latest/). Dù bạn đang xây dựng một nền tảng AI/machine learning (ML) mới hay tối ưu hóa quy trình hiện có, những mẫu triển khai này sẽ giúp bạn tăng tốc quá trình phát triển trong khi vẫn duy trì hiệu quả vận hành.

## Tính năng chính khiến EKS Auto Mode lý tưởng cho AI/ML workloads

Trong phần này, chúng ta sẽ tìm hiểu chi tiết hơn về các tính năng chuyên biệt cho GPU được cấu hình sẵn và sẵn sàng sử dụng trong EKS Auto Mode clusters. Những khả năng này cũng có sẵn trong môi trường Amazon EKS tự quản lý, nhưng thường cần phải thiết lập và tinh chỉnh. Tuy nhiên, với EKS Auto Mode các tính năng này được kích hoạt và cấu hình sẵn ngay từ đầu.

**Tự động mở rộng linh hoạt với Karpenter:** EKS Auto Mode bao gồm một phiên bản được quản lý của [Karpenter ](https://karpenter.sh/)cung cấp một công cụ mã nguồn mở tương thích với[ Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/) có kích thước phù hợp, bao gồm cả các tùy chọn tăng tốc bằng GPU, dựa trên yêu cầu của pod. Giải pháp này hỗ trợ cơ chế mở rộng tức thời và cho phép bạn tùy chỉnh hành vi cấp phát tài nguyên để tối ưu theo chi phí, hiệu năng hoặc vị trí triển khai của instance. EKS Auto Mode hỗ trợ sẵn[ ](https://docs.aws.amazon.com/eks/latest/userguide/automode-learn-instances.html)[các loại và kích thước instance](https://docs.aws.amazon.com/eks/latest/userguide/automode-learn-instances.html) được định nghĩa trước, đồng thời cung cấp [các nhãn và thuộc tính hạn chế (taints)](https://docs.aws.amazon.com/eks/latest/userguide/create-node-pool.html#auto-supported-labels) để kiểm soát lịch trình triển khai một cách chính xác.

**Tự động xử lý sự cố GPU:** EKS Auto Mode bao gồm [Node Monitoring Agent (NMA) và Node Auto Repair](https://aws.amazon.com/blogs/containers/amazon-eks-introduces-node-monitoring-and-auto-repair-capabilities/), cho phép phát hiện lỗi GPU và tự động khởi động quá trình khôi phục sau 10 phút kể từ khi phát hiện sự cố. Hệ thống sửa chữa sẽ cô lập node bị ảnh hưởng và thực hiện khởi động lại hoặc thay thế node đó, trong khi vẫn tuân thủ Pod Disruption Budgets. Các công cụ giám sát GPU, như [DCGM-Exporter](https://github.com/NVIDIA/dcgm-exporter) cho NVIDIA hoặc [Neuron Monitor](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/tools/neuron-sys-tools/neuron-monitor-user-guide.html) cho Amazon Web Services (AWS) [Inferentia](https://aws.amazon.com/ai/machine-learning/inferentia/) và AWS [Trainium](https://aws.amazon.com/ai/machine-learning/trainium/), được cài đặt sẵn và tích hợp với NMA giúp giám sát tình trạng phần cứng ở cấp độ thiết bị.

**Amazon EKS-optimized AMIs cho instances tăng tốc (accelerated instances):** EKS Auto Mode cho phép bạn tạo một [Karpenter NodePool](https://karpenter.sh/docs/concepts/nodepools/) sử dụng GPU instance. Hơn nữa, khi workload yêu cầu GPU, hệ thống sẽ tự động khởi tạo Amazon Machine Image (AMI) [Bottlerocket](https://aws.amazon.com/bottlerocket/) Accelerated - mà không cần cấu hình thủ công AMI IDs, mẫu khởi chạy (launch templates), hoặc software components. Các AMIs này được cài đặt sẵn driver cần thiết, runtimes, và plugins, dù bạn đang sử dụng GPU của NVIDIA hay bộ tăng tốc AWS Inferentia và Trainium, nhờ vậy AI workloads của bạn sẵn sàng chạy ngay mặc định.

Tổng hợp lại, các tính năng này loại bỏ phần lớn công việc phức tạp trong việc cấu hình và vận hành hạ tầng GPU, giúp các nhóm kỹ thuật tập trung vào việc xây dựng, mở rộng và triển khai các tác vụ AI/ML mà không cần phải trở thành chuyên gia Kubernetes.

## Hướng dẫn thực hành

Trong phần này, bạn sẽ thực hành triển khai một mô hình LLM mã nguồn mở trên EKS Auto Mode có hỗ trợ GPU. Bạn sẽ tạo cluster, cấu hình một GPU NodePool, triển khai mô hình, và gửi một truy vấn thử nghiệm (test prompt), tất cả chỉ với cấu hình đơn giản.

### Yêu cầu

Để bắt đầu, hãy đảm bảo rằng bạn đã tải và cài đặt cấu hình sau:

- [AWS Command Line Interface (AWS CLI](https://aws.amazon.com/cli/) (v2.27.11 hoặc phiên bản gần nhất)

- [kubectl](https://kubernetes.io/docs/reference/kubectl/)

- [eksctl](https://eksctl.io/) (v0.195.0 hoặc phiên bản gần nhất)

- [jq](https://jqlang.org/)

### Cài đặt biến môi trường

Cấu hình các biến môi trường, thay thế các giá trị mẫu bằng thông tin của bạn:
```
export CLUSTER_NAME=automode-gpu-blog-cluster
export AWS_REGION=us-west-2
```

### Cài đặt EKS Auto Mode cluster và chạy model

**Bước 1:** Tạo một EKS Auto Mode bằng `eksctl`

Bắt đầu tạo EKS cluster của bạn với Auto Mode có sẵn bằng cách chạy dòng dưới trong command:
```
eksctl create cluster --name=$CLUSTER_NAME --region=$AWS_REGION --enable-auto-mode
```

Quá trình này có thể mất vài phút để hoàn thánh. Sau khi hoàn tất, `eksctl` tự động cập nhật kubeconfig và chuyển sang cluster vừa được tạo. Để xác minh rằng cluster đang hoạt động, sử dụng lệnh sau:

```
kubectl get pods --all-namespaces
```

Ví dụ về output:
```
NAMESPACE     NAME                                  READY   STATUS    RESTARTS   AGE
kube-system   metrics-server-6d67d68f67-7x4tg       1/1     Running   0          3m
kube-system   metrics-server-6d67d68f67-l4xv6       1/1     Running   0          3m
```

Bạn sẽ không thấy các thành phần như VPC CNI, `kube-proxy`, Karpenter và `CoreDNS` trong danh sách pod. Trong EKS Auto Mode, AWS vận hành các thành phần này trên lớp hạ tầng được quản lý hoàn toàn, cùng với control plane của Amazon EKS.

**Bước 2:** Tạo một GPU NodePool với Karpenter

Triển khai một GPU NodePool được thiết kế để chạy ML models. Áp dụng NodePool manifest:

```
cat << EOF | kubectl apply -f -
apiVersion: karpenter.sh/v1
kind: NodePool
metadata:
  name: gpu-node-pool
spec:
  template:
    metadata:
      labels:
        type: karpenter
        NodeGroupType: gpu-node-pool
    spec:
      nodeClassRef:
        group: eks.amazonaws.com
        kind: NodeClass
        name: default
      taints:
        - key: nvidia.com/gpu
          value: Exists
          effect: NoSchedule
      requirements:
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["spot", "on-demand"]
        - key: eks.amazonaws.com/instance-category
          operator: In
          values: ["g"]
        - key: eks.amazonaws.com/instance-generation
          operator: Gt
          values: ["4"]
        - key: kubernetes.io/arch
          operator: In
          values: ["amd64"]
  limits:
    cpu: 100
    memory: 100Gi
EOF
```

NodePool này nhắm tới các instance EC2 dùng GPU thuộc họ `g `với thế hệ từ 4 trở đi, chẳng hạn G5 và G6e. Những instance này này cung cấp GPU NVIDIA mạnh cùng kết nối mạng băng thông cao, nên rất phù hợp cho các workload suy luận ML đòi hỏi cao và generative AI. Taint được áp dụng đảm bảo chỉ các pod đủ điều kiện dùng GPU mới được lên lịch trên các node này, duy trì cô lập tài nguyên hiệu quả. Việc cho phép đồng thời hai loại On-Demand và Spot mang lại cho EKS Auto Mode tính linh hoạt để tối ưu chi phí trong khi vẫn đảm bảo hiệu năng.

**Kiểm tra NodePool:**
```
kubectl get nodepools
```
**Mẫu output:**
```
NAME              NODECLASS   NODES   READY   AGE
general-purpose   default     0       True    15m
gpu-node-pool     default     0       True    8s
system            default     2       True    15m
```
`gpu-node-pool`  được khởi tạo với 0 node. Để kiểm tra, chạy lệnh sau:

```
kubectl get nodes -o custom-columns=NAME:.metadata.name,READY:"status.conditions[?(@.type=='Ready')].status",OS-IMAGE:.status.nodeInfo.osImage,INSTANCE-TYPE:.metadata.labels.'node\.kubernetes\.io/instance-type',LIFECYCLE:.metadata.labels.'karpenter\.sh/capacity-type'
```
**Mẫu output:**
```
NAME                  READY     OS-IMAGE                                                              INSTANCE-TYPE   LIFECYCLE
i-0319343e8ad4c5f14   True      Bottlerocket (EKS Auto, Standard) 2025.7.18 (aws-k8s-1.32-standard)   c6g.large       on-demand
i-0a3ff5bfd7be551e2   True      Bottlerocket (EKS Auto, Standard) 2025.7.18 (aws-k8s-1.32-standard)   c6g.large       on-demand
```

EKS Auto Mode khởi chạy hai `c6g` instances sử dụng Bottlerocket AMI không tăng tốc (non-accelerated) (`aws-k8s-1.32-standard`), các instance này chỉ sử dụng CPU (CPU-only) và được dùng để chạy dịch vụ metrics server.

**Bước 3.** Triển khai mô hình gpt-oss-20b sử dụng vLLM

[vLLM](https://docs.vllm.ai/en/latest/)[ ](https://docs.vllm.ai/en/latest/)là công cụ suy luận (inference engine) mã nguồn mở hiệu năng cao được tối ưu hóa cho các mô hình ngôn ngữ lớn (LLMs). Tệp YAML triển khai container image độc lập (model-agnostic) `vllm``/``vllm``-openai:``gptoss`. Trong ví dụ này, ta chỉ định `openai/gpt-oss-20b` làm mô hình mà vLLM sẽ phục vụ.

```
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpt-oss-20b
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vllm-gptoss-20b
  template:
    metadata:
      labels:
        app: vllm-gptoss-20b
    spec:
      tolerations:
        - key: nvidia.com/gpu
          operator: Exists
          effect: NoSchedule
      containers:
      - name: inference-server
`        image: ``vllm``/``vllm``-openai:``gptoss`

        ports:
        - containerPort: 8000
        resources:
          limits:
            nvidia.com/gpu: 1
        command: [ "vllm", "serve" ]
        args:
        - openai/gpt-oss-20b
        - --gpu-memory-utilization=0.90
        - --tensor-parallel-size=1
        - --max-model-len=20000
        env:
        - name: VLLM_ATTENTION_BACKEND
          value: "TRITON_ATTN_VLLM_V1"
        - name: PORT
          value: "8000"
        volumeMounts:
        - mountPath: /dev/shm
          name: dshm
      volumes:
      - name: dshm
        emptyDir:
          medium: Memory
---
apiVersion: v1
kind: Service
metadata:
  name: gptoss-service
spec:
  selector:
    app: vllm-gptoss-20b
  ports:
  - port: 8000
    targetPort: 8000
  type: ClusterIP
EOF
```

Triển khai này sử dụng toleration (miễn nhiễm)  `nvidia.com/gpu`, tương ứng với taint trên GPU NodePool. Ban đầu, chưa có node GPU nào tồn tại, vì vậy pod sẽ ở trạng thái  `Pending`. Karpenter sẽ phát hiện pod không thể được lên lịch và tự động khởi tạo một node GPU mới. Khi instance sẵn sàng, pod được lên lịch và chuyển sang trạng thái `ContainerCreating`, tại thời điểm đó hệ thống bắt đầu tải (pull) container image của  `vllm`. Sau khi container image được tải và giải nén hoàn tất, container chuyển sang trạng thái `Running`.

Chờ đến khi pod chuyển sang trạng thái  `Running`. Để theo dõi các sự kiện của pod, dùng lệnh sau:
```
kubectl get pods -l app=vllm-gptoss-20b -w
```
**Mẫu output:**
```
NAME                            READY   STATUS    RESTARTS   AGE
gpt-oss-20b-7dc7f7658d-8xsbm    1/1     Running   0          5m
```

Để kiểm tra pod event dùng lệnh:
```
kubectl describe pod -l app=vllm-gptoss-20b
```

**Mẫu output:**

```
Events:
  Type     Reason            Age    From                   Message
  ----     ------            ----   ----                   -------
  Warning  FailedScheduling  2m33s  default-scheduler      0/1 nodes are available: 1 node(s) had untolerated taint {CriticalAddonsOnly: }. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
  Normal   Nominated         2m32s  eks-auto-mode/compute  Pod should schedule on: nodeclaim/gpu-node-pool-c4bb5
  Normal   Scheduled         95s    default-scheduler      Successfully assigned default/gpt-oss-20b-68b49c7b44-2r7gr to i-05a572a3bbed6669f
  Normal   Pulling           89s    kubelet                Pulling image "public.ecr.aws/deep-learning-containers/vllm:0.11.2-gpu-py312-ec2-soci"
  Normal   Pulled            1s     kubelet                Successfully pulled image "public.ecr.aws/deep-learning-containers/vllm:0.11.2-gpu-py312-ec2-soci" in 1m27.666s (1m27.666s including waiting). Image size: 14221606791 bytes.
  Normal   Created           1s     kubelet                Created container: inference-server
  Normal   Started           1s     kubelet                Started container inference-server
```

Mất vài phút để pod chuyển sang trạng thái `Running`. Trong ví dụ trên, Karpenter đã khởi tạo instance và lên lịch cho pod chưa đầy một phút. Thời gian còn lại dùng để tải image `vllm` từ internet, có dung lượng khoảng 14 GB.

Khi container chuyển sang trạng thái `Running`, trọng số của mô hình (model weights) bắt đầu được nạp vào bộ nhớ GPU, quá trình này mất vài phút để hoàn tất. Xem logs để theo dõi tiến trình:
```
kubectl logs -l app=vllm-gptoss-20b -f
```

Khi mô hình được tải xong, bạn sẽ thấy output giống như sau:

```
INFO 08-22 22:26:52 [launcher.py:37] Route: /rerank, Methods: POST
INFO 08-22 22:26:52 [launcher.py:37] Route: /v1/rerank, Methods: POST
INFO 08-22 22:26:52 [launcher.py:37] Route: /v2/rerank, Methods: POST
INFO 08-22 22:26:52 [launcher.py:37] Route: /scale_elastic_ep, Methods: POST
INFO 08-22 22:26:52 [launcher.py:37] Route: /is_scaling_elastic_ep, Methods: POST
INFO 08-22 22:26:52 [launcher.py:37] Route: /invocations, Methods: POST
INFO 08-22 22:26:52 [launcher.py:37] Route: /metrics, Methods: GET
INFO:     Started server process [1]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```
Sau khi áp dụng manifest, Karpenter đã khởi tạo một GPU instance đáp ứng các ràng buộc (constraints) được định nghĩa trong NodePool. Để xem instance nào đã được khởi tạo, chạy lệnh sau:

```
kubectl get nodes -o custom-columns=NAME:.metadata.name,READY:"status.conditions[?(@.type=='Ready')].status",OS-IMAGE:.status.nodeInfo.osImage,INSTANCE-TYPE:.metadata.labels.'node\.kubernetes\.io/instance-type',LIFECYCLE:.metadata.labels.'karpenter\.sh/capacity-type'
```
**Mẫu output:**
```
NAME                  READY     OS-IMAGE                                                              INSTANCE-TYPE   LIFECYCLE
i-0319343e8ad4c5f14   True      Bottlerocket (EKS Auto, Standard) 2025.7.18 (aws-k8s-1.32-standard)   c6g.large       on-demand
i-0a3ff5bfd7be551e2   True      Bottlerocket (EKS Auto, Standard) 2025.7.18 (aws-k8s-1.32-standard)   c6g.large       on-demand
i-029d33a1259f77564   True      Bottlerocket (EKS Auto, Nvidia) 2025.7.25 (aws-k8s-1.32-nvidia)       g6e.xlarge      spot
```

Trong trường hợp này Karpenter xác định rằng [G6e](https://aws.amazon.com/ec2/instance-types/g6e/) xlarge spot instance là loại instance có chi phí hiệu quả và tuân thủ các ràng buộc được định nghĩa trong NodePool.

**Bước 5.** Kiểm tra model endpoints

Đầu tiên, thực thi một port forward đến `gptoss``-service` service sử dụng kubectl:
```
kubectl port-forward service/gptoss-service 8000:8000
```

Trong một terminal khác, gửi một test prompt bằng cách sử dụng `curl`:
```
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "openai/gpt-oss-20b",
    "messages": [
      {
        "role": "user",
        "content": "What is machine learning?"
      }
    ],
    "temperature": 0.7,
    "max_tokens": 100
  }' | jq -r '.choices[0].message.content'
```

**Mẫu output:**

```
**Machine learning (ML)** is a branch of computer science that gives computers the ability to learn from data, identify patterns, and make decisions or predictions without being explicitly programmed to perform each specific task.
```

Thiết lập này cho phép bạn test và tương tác với inference server của mình mà không cần công khai nó ra bên ngoài. Để làm cho nó có thể truy cập được bởi các ứng dụng hoặc người dùng khác, bạn có thể cập nhật service type thành  `LoadBalancer`, cho phép truy cập từ bên ngoài hoặc trong VPC của bạn. Nếu công khai service, hãy đảm bảo triển khai các kiểm soát truy cập phù hợp như authentication, authorization, và rate limiting.

**Bước 5.** Dọn dẹp tài nguyên

Khi bạn đã hoàn thành các thử nghiệm của mình, bạn phải dọn dẹp các tài nguyên đã tạo để tránh phát sinh chi phí liên tục. Để xóa cluster và tất cả các tài nguyên liên quan được quản lý bởi EKS Auto Mode, hãy chạy lệnh sau:
```
eksctl delete cluster --name=$CLUSTER_NAME --region=$AWS_REGION
```

Lệnh này sẽ xóa toàn bộ EKS cluster cùng với control plane, data plane nodes, NodePools, và tất cả các tài nguyên được quản lý bởi EKS Auto Mode.

## Cách giảm thời gian khởi động “cold start” cho AI inference workloads

Như bạn đã thấy trong phần trước, quá trình này mất vài phút để container image được tải xuống, model được lấy về, và weights được nạp vào GPU memory. Độ trễ này thường được gây ra bởi container image có kích thước lớn (trên 17GB trong trường hợp này), việc tải model từ nguồn bên ngoài, và thời gian cần thiết để nạp model vào bộ nhớ, tất cả đều làm tăng độ trễ khi pod khởi động và trong các sự kiện mở rộng. Trong môi trường sản xuất, đặc biệt khi chạy inference ở quy mô lớn, bạn phải sử dụng Kubernetes autoscaling và giảm thiểu thời gian khởi động này để đảm bảo mở rộng nhanh và phản hồi kịp thời. Trong phần này, chúng ta sẽ đi qua các kỹ thuật nhằm tối ưu hóa thời gian khởi động model và giảm độ trễ khởi động lạnh (cold start delays).

**Lưu trữ vLLM container image trong Amazon ECR và sử dụng VPC endpoint:** Pulling container images từ các public registries trên internet gây ra độ trễ trong quá trình pod khởi động, đặc biệt khi image có kích thước lớn hoặc băng thông mạng bị giới hạn. Để giảm chi phí xử lý này:

- **Lưu trữ container image của bạn trong** [Amazon Elastic Container Registry (Amazon ECR)](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html), một container registry được quản lý hoàn toàn khả dụng theo khu vực và được tối ưu hóa để dùng với Amazon EKS.

- **Cấu hình** [Amazon ECR VPC endpoint ](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-ecr.html)để các nodes pull images thông qua AWS backbone thay vì public internet.

**Các mô hình được tải sẵn (prefetch model artifacts) bằng cách sử dụng các tùy chọn của AWS Storage:** Để giảm thời gian khởi động do việc tải xuống và load model từ [Hugging Face](https://huggingface.co/) gây ra, hãy lưu trữ các model artifacts trong một tùy chọn lưu trữ AWS hỗ trợ truy cập đồng thời và đọc với thông lượng cao trên nhiều nodes và [AWS Availability Zones (](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/)[AZs](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/)[)](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/). Điều này là thiết yếu khi nhiều bản sao của inference service của bạn,có thể đang chạy trên các nodes hoặc AZs khác nhau, cần đọc cùng một model weights đồng thời. Việc sử dụng shared storage giúp tránh phải tải xuống và lưu trữ các bản sao trùng lặp của model cho mỗi pod hoặc node.

- [Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) **với **[Mountpoint](https://github.com/awslabs/mountpoint-s3) **và **[S3 Express One Zone](https://docs.aws.amazon.com/AmazonS3/latest/userguide/directory-bucket-high-performance.html#s3-express-one-zone): Express One Zone lưu trữ dữ liệu trong một AZ, mặc dù dữ liệu đó có thể được truy cập từ các AZ khác trong cùng một Region. Đây là tùy chọn lưu trữ có chi phí thấp nhất và dễ thiết lập nhất. Giải pháp này lý tưởng cho các công việc suy luận tổng quát nơi yêu cầu hiệu năng ở mức vừa phải và độ trực tiếp là yếu tố chính. Để đạt kết quả tốt nhất, hãy cấu hình một [VPC endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html) nhằm đảm bảo rằng lưu lượng luôn nằm trong mạng AWS.

- [Amazon Elastic File System (Amazon EFS)](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html): Là một dịch vụ đa vùng sẵn có (natively multi-AZ service), tự động sao chép dữ liệu giữa các AZs. Amazon EFS là một hệ thống chia sẻ tệp dễ sử dụng, mang lại sự cân bằng tốt giữa chi phí, độ trễ và thông lượng. Phù hợp cho các tải công việc cần truy cập mô hình ổn định từ nhiều AZs, với tính sẵn sàng cao được tích hợp sẵn.

- [Amazon FSx for Lustre](https://docs.aws.amazon.com/fsx/latest/LustreGuide/what-is.html): Được triển khai trong một AZ duy nhất và có thể truy cập từ các AZ khác trong cùng một VPC. Dịch vụ này cung cấp tùy chọn hiệu năng cao nhất cho shared storage.Mặc dù FSx for Lustre có thể có chi phí lưu trữ cao hơn, nhưng tốc độ tải model weights của nó có thể giảm thời gian trống của GPU (GPU idle time), thường cân bằng lại chi phí đồng thời mang lại hiệu suất tải mô hình nhanh nhất.

Tách biệt model artifacts khỏi container images, lưu trữ containers của bạn trong Amazon ECR, và lựa chọn storage backend phù hợp cho models của bạn,chẳng hạn như Amazon S3 Mountpoint, Amazon EFS, hoặc Amazon FSx for Lustre cho phép bạn giảm đáng kể thời gian khởi động và cải thiện khả năng phản hồi của các tải công việc suy luận. Để khám phá thêm các chiến lược nhằm tối ưu hóa thời gian khởi động của container và model trên Amazon EKS, hãy tham khảo tài liệu  [AI on Amazon EKS guidance](https://awslabs.github.io/ai-on-eks/docs/guidance/container-startup-time).

## Kết luận

Amazon EKS Auto Mode đơn giản hóa việc chạy các công việc suy luận AI sử dụng GPU bằng cách xử lý các tác vụ như khởi tạo cluster, node scaling, và cấu hình GPU thay cho bạn. Tự động mở rộng động thông qua Karpenter, AMIs được cấu hình sẵn, cùng với giám sát và khôi phục GPU tích hợp cho phép bạn triển khai model nhanh hơn - mà không cần phải cấu hình hoặc duy trì hạ tầng nền tảng.

Để tìm hiểu thêm về việc chạy các inference workloads trên EKS Auto Mode, dưới đây là một vài bước tiếp theo:

- **Tìm hiểu thêm:** Truy cập [EKS Auto Mode documentation](https://docs.aws.amazon.com/eks/latest/userguide/automode.html) để xem đầy đủ capabilities, các loại instance được hỗ trợ, và tùy chọn cấu hình. Bạn cũng có thể xem [EKS Auto Mode post](https://aws.amazon.com/blogs/containers/getting-started-with-amazon-eks-auto-mode/) để giới thiệu thực hành.

- **Trải nghiệm thực tế:** Tham gia các buổi hội thảo ảo có giảng viên hướng dẫn thuộc chuỗi Amazon EKS series, trong đó có các session chuyên biệt về Auto Mode và AI inference.

- **Khám phá các phương pháp tốt nhất:** Xem Amazon [EKS best practices guide for AI/ML workloads](https://docs.aws.amazon.com/eks/latest/best-practices/aiml.html).

- **Lập kế hoạch về khả năng mở rộng và chi phí: **Nếu bạn đang chạy LLMs hoặc GPU workloads có nhu cầu cao, hãy liên hệ với AWS account team của bạn để được hướng dẫn về định giá, khuyến nghị tối ưu kích thước, và hỗ trợ lập kế hoạch. AWS gần đây [đã thông báo giảm giá tới 45%](https://aws.amazon.com/blogs/aws/announcing-up-to-45-price-reduction-for-amazon-ec2-nvidia-gpu-accelerated-instances/) cho các instance được tăng tốc bằng NVIDIA GPU, bao gồm P4d, P4de, và P5, vì vậy đây là thời điểm thích hợp để đánh giá các tùy chọn của bạn.

Các công cụ và hướng dẫn này cho phép bạn chạy các tải công việc suy luận ở quy mô lớn với ít nỗ lực hơn và tự tin hơn.

## Các tác giả
<img src="/images/3-Blog/3.2-Blog2/shivam-bio.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

**Shivam Dubey** là Kiến trúc sư Giải pháp Chuyên biệt (Specialist Solutions Architect) tại Amazon Web Services (AWS), nơi anh hỗ trợ khách hàng xây dựng các giải pháp mở rộng quy mô và tích hợp Trí tuệ nhân tạo trên Amazon EKS. Anh có niềm đam mê với các công nghệ mã nguồn mở và vai trò của chúng trong kiến trúc điện toán đám mây hiện đại. Ngoài công việc, Shivam yêu thích leo núi, tham quan các công viên quốc gia và khám phá những thể loại âm nhạc mới.
<div style="clear: both;"></div>

<img src="/images/3-Blog/3.2-Blog2/Bharath-Bio.jpg" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

**Bharath Gajendran** là Quản lý Tài khoản Kỹ thuật tại Amazon Web Services (AWS), nơi anh hỗ trợ khách hàng thiết kế và vận hành các khối lượng công việc có khả năng mở rộng cao, tối ưu chi phí và chịu lỗi hiệu quả bằng cách tận dụng các dịch vụ của AWS. Anh có niềm đam mê với Amazon EKS và các công nghệ mã nguồn mở, đồng thời chuyên về việc giúp các tổ chức triển khai và mở rộng quy mô các khối lượng công việc AI trên EKS.
<div style="clear: both;"></div>

<img src="/images/3-Blog/3.2-Blog2/Christina-Andonov.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

**Christina Andonov** là Kiến trúc sư Giải pháp Chuyên biệt Cấp cao tại Amazon Web Services (AWS), nơi cô hỗ trợ khách hàng triển khai các khối lượng công việc AI trên Amazon EKS bằng cách ứng dụng các công cụ mã nguồn mở. Cô có niềm đam mê đặc biệt với Kubernetes và được biết đến với khả năng truyền đạt những khái niệm phức tạp một cách dễ hiểu và trực quan.
