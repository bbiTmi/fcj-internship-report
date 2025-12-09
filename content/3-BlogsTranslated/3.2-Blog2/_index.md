---
title: "Blog 2"
date: ""
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---
# How to run AI model inference with GPUs on Amazon EKS Auto Mode

AI model inference using GPUs is becoming a core part of modern applications, powering real-time recommendations, intelligent assistants, content generation, and other latency-sensitive AI features. Kubernetes has become the orchestrator of choice for running inference workloads, and organizations want to use its capabilities while still maintaining a strong focus on rapid innovation and time-to-market. But here’s the challenge: while teams see the value of Kubernetes for its dynamic scaling and efficient resource management, they often get slowed down by the need to learn Kubernetes concepts, manage cluster configurations, and handle security updates. This shifts focus away from what matters most: deploying and optimizing AI models. That is where [Amazon Elastic Kubernetes Service (Amazon EKS) Auto Mode](https://docs.aws.amazon.com/eks/latest/userguide/automode.html) comes in. EKS Auto Mode Automates node creation, manages [core capabilities](https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html#addon-consider-auto), and handles upgrades and security patching. In turn, this enables to run your inference workloads without the operational overhead.

In this post, we show you how to swiftly deploy inference workloads on EKS Auto Mode. We also demonstrate key features that streamline GPU management, show best practices for model deployment, and walk through a practical example by deploying [open weight models from OpenAI](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-openai.html) using [vLLM](https://docs.vllm.ai/en/latest/). Whether you’re building a new AI/machine learning (ML) platform or optimizing existing workflows, these patterns help you accelerate development while maintaining operational efficiency.


## Key features that make EKS Auto Mode ideal for AI/ML workloads

In this section, we take a closer look at the GPU-specific features that come pre-configured and ready to use with an EKS Auto Mode cluster. These capabilities are also available in self-managed Amazon EKS environments, but they typically need manual setup and tuning. However, EKS Auto Mode has them enabled and configured out of the box.

Dynamic autoscaling with Karpenter: EKS Auto Mode includes a managed version of open source conformant [Karpenter](https://karpenter.sh/) that provisions right-sized [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/) instances, such as GPU‑accelerated options, based on pod requirements. It supports just-in-time scaling and allows you to configure provisioning behavior so that you can optimize for cost, performance, or instance placement. EKS Auto Mode supports a predefined set of [instance types and sizes](https://docs.aws.amazon.com/eks/latest/userguide/automode-learn-instances.html), along with [node labels and taints](https://docs.aws.amazon.com/eks/latest/userguide/create-node-pool.html#auto-supported-labels) for scheduling control.

Automatic GPU failure handling: EKS Auto Mode includes [Node Monitoring Agent (NMA) and Node Auto Repair](https://aws.amazon.com/blogs/containers/amazon-eks-introduces-node-monitoring-and-auto-repair-capabilities/), which detect GPU failures and initiate automated recovery 10 minutes after detection. The repair process cordons the affected node and either reboots or replaces it, while respecting Pod Disruption Budgets. GPU telemetry tools, either [DCGM-Exporter](https://github.com/NVIDIA/dcgm-exporter) for NVIDIA or [Neuron Monitor](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/tools/neuron-sys-tools/neuron-monitor-user-guide.html) for Amazon Web Services (AWS) [Inferentia](https://aws.amazon.com/ai/machine-learning/inferentia/) and AWS [Trainium](https://aws.amazon.com/ai/machine-learning/trainium/), are pre-installed and integrated with NMA for device-level health monitoring.

Amazon EKS-optimized AMIs for accelerated instances: EKS Auto Mode allows you to create a [Karpenter NodePool](https://karpenter.sh/docs/concepts/nodepools/) using GPU instance types. Furthermore, when a workload requests a GPU, it automatically launches the appropriate [Bottlerocket](https://aws.amazon.com/bottlerocket/) Accelerated Amazon Machine Image (AMI)—with no need to configure AMI IDs, launch templates, or software components. These AMIs come pre-installed with the necessary drivers, runtimes, and plugins, whether you’re using NVIDIA GPUs or AWS Inferentia and Trainium, so that your AI workloads are ready to run by default.

Together, these features remove the heavy lifting of configuring and operating GPU infrastructure, so that teams can focus on building, scaling, and running AI/ML workloads without becoming Kubernetes experts.


## Walkthrough

In this section, you’ll walk through deploying an open-source large language model (LLM) on a GPU-enabled EKS Auto Mode cluster. You’ll create the cluster, configure a GPU NodePool, deploy the model, and send a test prompt, all with minimal setup.


### **Prerequisites**

To get started, make sure that you have the following prerequisites installed and configured:



* [AWS Command Line Interface (AWS CLI](https://aws.amazon.com/cli/)) (v2.27.11 or later)
* [kubectl](https://kubernetes.io/docs/reference/kubectl/)
* [eksctl](https://eksctl.io/) (v0.195.0 or later)
* [jq](https://jqlang.org/)


### **Set up environment variables**

Configure the following environment variables, replacing the placeholder values as appropriate for your setup:


```
export CLUSTER_NAME=automode-gpu-blog-cluster
export AWS_REGION=us-west-2
```



### **Set up EKS Auto Mode cluster and run a model**

**Step 1:** Create an EKS Auto Mode using `eksctl`

Begin by creating your EKS cluster with Auto Mode enabled by running the following command:


```
eksctl create cluster --name=$CLUSTER_NAME --region=$AWS_REGION --enable-auto-mode
```


This process takes a few minutes to complete. After completion, `eksctl` automatically updates your kubeconfig and targets your newly created cluster. To verify that the cluster is operational, use the following:


```
kubectl get pods --all-namespaces
```


**Sample output:**


```
NAMESPACE     NAME                                  READY   STATUS    RESTARTS   AGE
kube-system   metrics-server-6d67d68f67-7x4tg       1/1     Running   0          3m
kube-system   metrics-server-6d67d68f67-l4xv6       1/1     Running   0          3m
```


You won’t see components such as VPC CNI, `kube-proxy`, Karpenter, and `CoreDNS` in the pod list. In EKS Auto Mode, AWS runs these components on the fully managed infrastructure layer, alongside the Amazon EKS control plane.

**Step 2:** Create a GPU NodePool with Karpenter

Deploy a GPU NodePool tailored to run ML models. Apply the following NodePool manifest:


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


This NodePool targets GPU-based EC2 instances in the `g` category with a generation greater than four, such as G5 and G6e instances. These instance families offer powerful NVIDIA GPUs and high-bandwidth networking, making them well-suited for demanding ML inference and generative AI workloads. The applied taint makes sure that only GPU-eligible pods are scheduled on these nodes, maintaining efficient resource isolation. Allowing both On-Demand and Spot capacity types gives EKS Auto Mode the flexibility to optimize for cost while maintaining performance.

**Validate the NodePool:**


```
kubectl get nodepools
```


**Sample output:**


```
NAME              NODECLASS   NODES   READY   AGE
general-purpose   default     0       True    15m
gpu-node-pool     default     0       True    8s
system            default     2       True    15m
```


The `gpu-node-pool` is created with zero nodes initially. To inspect available nodes, use:


```
kubectl get nodes -o custom-columns=NAME:.metadata.name,READY:"status.conditions[?(@.type=='Ready')].status",OS-IMAGE:.status.nodeInfo.osImage,INSTANCE-TYPE:.metadata.labels.'node\.kubernetes\.io/instance-type',LIFECYCLE:.metadata.labels.'karpenter\.sh/capacity-type'
```


**Sample output:**


```
NAME                  READY     OS-IMAGE                                                              INSTANCE-TYPE   LIFECYCLE
i-0319343e8ad4c5f14   True      Bottlerocket (EKS Auto, Standard) 2025.7.18 (aws-k8s-1.32-standard)   c6g.large       on-demand
i-0a3ff5bfd7be551e2   True      Bottlerocket (EKS Auto, Standard) 2025.7.18 (aws-k8s-1.32-standard)   c6g.large       on-demand
```


EKS Auto Mode runs two `c6g` instances using the non-accelerated Bottlerocket AMI variant (`aws-k8s-1.32-standard`), which are CPU-only and used for running metrics server.

**Step 3.** Deploy the gpt-oss-20b model using vLLM

[vLLM](https://docs.vllm.ai/en/latest/) is a high-throughput, open source inference engine optimized for large language models (LLMs). The following YAML deploys the `vllm` container image, which is model-agnostic. In this example, we specify `openai/gpt-oss-20b` as the model for vLLM to serve.


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
        image: public.ecr.aws/deep-learning-containers/vllm:0.11.2-gpu-py312-ec2-soci
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
          value: "TRITON_ATTN"
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


This deployment uses a toleration for `nvidia.com/gpu`, matching the taint on your GPU NodePool. Initially, no GPU nodes are present, so the pod enters the `Pending` state. Karpenter detects the unschedulable pod and automatically provision a GPU node. When the instance is ready, the pod is scheduled and transitions to the `ContainerCreating` state, at which point it begins pulling the `vllm` container image. When the container image is pulled and unpacked, the container enters the `Running` state.

Wait for the pod to show `Running`. To monitor pod events, use the following:


```
kubectl get pods -l app=vllm-gptoss-20b -w
```


**Sample output:**


```
NAME                            READY   STATUS    RESTARTS   AGE
gpt-oss-20b-7dc7f7658d-8xsbm    1/1     Running   0          5m
```


To check the pod events use the following:


```
kubectl describe pod -l app=vllm-gptoss-20b
```


**Sample output:**


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


It may take a few minutes for the pod to be in the `Running` state. In the preceding example, Karpenter provisioned the instance and scheduled the pod in under a minute. The remaining time was spent downloading the `vllm` image over the internet, which is roughly 14 GB in size.

When the container is `Running`, the model weights start loading into GPU memory, which takes a few minutes. View logs to track the loading progress:


```
kubectl logs -l app=vllm-gptoss-20b -f
```


When the model has loaded, you should see output similar to the following:


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


After applying the manifest, Karpenter provisioned a GPU instance that satisfies the constraints defined in the NodePool. To see which instance was launched, run the following command:


```
kubectl get nodes -o custom-columns=NAME:.metadata.name,READY:"status.conditions[?(@.type=='Ready')].status",OS-IMAGE:.status.nodeInfo.osImage,INSTANCE-TYPE:.metadata.labels.'node\.kubernetes\.io/instance-type',LIFECYCLE:.metadata.labels.'karpenter\.sh/capacity-type'
```


**Sample output:**


```
NAME                  READY     OS-IMAGE                                                              INSTANCE-TYPE   LIFECYCLE
i-0319343e8ad4c5f14   True      Bottlerocket (EKS Auto, Standard) 2025.7.18 (aws-k8s-1.32-standard)   c6g.large       on-demand
i-0a3ff5bfd7be551e2   True      Bottlerocket (EKS Auto, Standard) 2025.7.18 (aws-k8s-1.32-standard)   c6g.large       on-demand
i-029d33a1259f77564   True      Bottlerocket (EKS Auto, Nvidia) 2025.7.25 (aws-k8s-1.32-nvidia)       g6e.xlarge      spot
```


In this case Karpenter determined that the [G6e](https://aws.amazon.com/ec2/instance-types/g6e/) xlarge spot instance is the most cost-efficient instance type that adheres to the constraints defined in the NodePool.

**Step 5.** Test the model endpoint

First, execute a port forward to the `gptoss-service` service using kubectl:


```
kubectl port-forward service/gptoss-service 8000:8000
```


In another terminal, send a test prompt using `curl`:


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


**Sample output:**


```
**Machine learning (ML)** is a branch of computer science that gives computers the ability to learn from data, identify patterns, and make decisions or predictions without being explicitly programmed to perform each specific task.
```


This setup allows you to test and interact with your inference server without exposing it externally. To make it accessible to other applications or users, you can update the service type to `LoadBalancer`, for either external access or within your VPC. If exposing the service, then make sure to implement appropriate access controls such as authentication, authorization, and rate limiting.

**Step 6.** Cleaning up

When you have finished your experiments, you must clean up the resources you created to avoid incurring ongoing charges. To delete the cluster and all associated resources managed by EKS Auto Mode, run the following command:


```
eksctl delete cluster --name=$CLUSTER_NAME --region=$AWS_REGION
```


This command removes the entire EKS cluster along with its control plane, data plane nodes, NodePools, and all resources managed by EKS Auto Mode.


## **Reducing model cold start time in AI inference workloads**

As you saw in the preceding section, it took a few minutes for the container image to download, the model to be fetched, and the weights to load into GPU memory. This delay is often caused by large container images (over 14 GB in this case), model downloads from external sources, and the time needed to load the model into memory, adding latency to pod startup and scaling events. In production scenarios, especially when running inference at scale, you must use Kubernetes autoscaling and minimize this startup time to make sure of fast, responsive scaling. In this section, we walk through techniques to optimize model startup time and reduce cold start delays.

Store vLLM container image in Amazon ECR and use a VPC endpoint: Pulling container images from public registries over the internet introduces latency during pod startup, especially when images are large or network bandwidth is constrained. To reduce this overhead:



* Store your container image in [Amazon Elastic Container Registry (Amazon ECR)](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html), a fully managed container registry that is regionally available and optimized for use with Amazon EKS.
* Configure an [Amazon ECR VPC endpoint ](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-ecr.html)so that nodes pull images over the AWS backbone rather than the public internet.

Prefetch model artifacts using AWS Storage options: To reduce the startup time introduced by model downloads and loading from [Hugging Face](https://huggingface.co/), store the model artifacts in an AWS storage option that supports concurrent access and high-throughput reads across multiple nodes and [AWS Availability Zones (AZs).](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) This is essential when multiple replicas of your inference service, possibly running across different nodes or AZs, need to read the same model weights simultaneously. Shared storage avoids the need to download and store duplicate copies of the model per pod or per node.



* [Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) with [Mountpoint](https://github.com/awslabs/mountpoint-s3) and [S3 Express One Zone](https://docs.aws.amazon.com/AmazonS3/latest/userguide/directory-bucket-high-performance.html#s3-express-one-zone): Express One Zone stores data in a single AZ, although it can be accessed from other AZs in the same Region. This is the lowest-cost storage option and most direct to set up. It is ideal for general-purpose inference workloads where performance requirements are moderate and directness is key. For best results, configure a [VPC endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html) to make sure that traffic stays within the AWS network.
* [Amazon Elastic File System (Amazon EFS)](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html): A natively multi-AZ service that automatically replicates data across AZs. Amazon EFS is an easy-to-use shared file system that offers a good balance of cost, latency, and throughput. It is suitable for workloads that need consistent access to models from multiple AZs with built-in high availability.
* [Amazon FSx for Lustre](https://docs.aws.amazon.com/fsx/latest/LustreGuide/what-is.html): Deployed in a single AZ and accessible from other AZs within the same VPC. It delivers the highest-performance option for shared storage. Although FSx for Lustre may have a higher storage cost, its speed in loading model weights can reduce overall GPU idle time, often balancing out the cost while providing the fastest model loading performance.

Separating model artifacts from container images, storing your containers in Amazon ECR, and choosing the right storage backend for your models, such as Amazon S3 Mountpoint, Amazon EFS, or Amazon FSx for Lustre allows you to significantly reduce startup time and improve the responsiveness of your inference workloads. To explore more strategies for optimizing container and model startup time on Amazon EKS, refer to the [AI on Amazon EKS guidance](https://awslabs.github.io/ai-on-eks/docs/guidance/container-startup-time).


## **Conclusion**

Amazon EKS Auto Mode streamlines running GPU-powered AI inference workloads by handling cluster provisioning, node scaling, and GPU configuration for you. Dynamic autoscaling through Karpenter, pre-configured AMIs, and built-in GPU monitoring and recovery enable you to deploy models faster—without needing to configure or maintain the underlying infrastructure.

To further explore running inference workloads on EKS Auto Mode, the following are a few next steps:



* Learn more: Visit the [EKS Auto Mode documentation](https://docs.aws.amazon.com/eks/latest/userguide/automode.html) for full capabilities, supported instance types, and configuration options. You can also check out the [EKS Auto Mode post](https://aws.amazon.com/blogs/containers/getting-started-with-amazon-eks-auto-mode/) for a hands-on introduction.
* Get hands-on experience: Join an instructor-led AWS virtual workshops from the [Amazon EKS series](https://aws-experience.com/emea/smb/events/series/get-hands-on-with-amazon-eks), featuring dedicated sessions on Auto Mode and AI inference.
* Explore best practices: Review the Amazon [EKS best practices guide for AI/ML workloads](https://docs.aws.amazon.com/eks/latest/best-practices/aiml.html).
* Plan for scale and costs: If you’re running LLMs or other high-demand GPU workloads, then connect with your AWS account team for pricing guidance, right-sizing recommendations, and planning support. AWS recently [announced up to a 45% price reduction](https://aws.amazon.com/blogs/aws/announcing-up-to-45-price-reduction-for-amazon-ec2-nvidia-gpu-accelerated-instances/) on NVIDIA GPU-accelerated instances, including the P4d, P4de, and P5, so it’s a good time to evaluate your options.

These tools and guidance allow you to run inference workloads at scale with less effort and more confidence.


---


### **About the authors**
<img src="/images/3-Blog/3.2-Blog2/shivam-bio.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

**Shivam Dubey** is a Specialist Solutions Architect at AWS, where he helps customers build scalable, AI-powered solutions on Amazon EKS. He is passionate about open-source technologies and their role in modern cloud-native architectures. Outside of work, Shivam enjoys hiking, visiting national parks, and exploring new genres of music.
<div style="clear: both;"></div>

<img src="/images/3-Blog/3.2-Blog2/Bharath-Bio.jpg" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

**Bharath Gajendran** is a Technical Account Manager at AWS, where he empowers customers to design and operate highly scalable, cost-effective, and fault-tolerant workloads utilizing AWS. He is passionate about Amazon EKS and open-source technologies, and specializes in enabling organizations to run and scale AI workloads on EKS.
<div style="clear: both;"></div>

<img src="/images/3-Blog/3.2-Blog2/Christina-Andonov.png" width="100" style="float: left; margin-right: 15px; margin-top:-10px;">

**Christina Andonov** is a Sr. Specialist Solutions Architect at AWS, helping customers run AI workloads on Amazon EKS with open source tools. She’s passionate about Kubernetes and known for making complex concepts easy to understand.