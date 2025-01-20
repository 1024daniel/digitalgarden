---
{"dg-publish":true,"permalink":"/AI/Infererence/nvidia推理框架/","noteIcon":"3"}
---

Sure, here's a diagram illustrating the relationships between Triton, TensorRT-LLM, TensorRT, and CUDA:

```txt unwrap
+--------------------+      +--------------------+      +-----------------------+      +--------------------+
|         CUDA        | -----> | TensorRT Parser     | -----> | TensorRT Optimizer   | -----> | TensorRT Builder     | -----> | TensorRT Runtime   |
+--------------------+      +--------------------+      +-----------------------+      +--------------------+
                             |                      |                             ^                         |
                             |                      |                             |                         |
                             +--------------------+                             +--------------------+
                                                 |
                                                 |
                             +--------------------+      +--------------------+
                             |  TensorRT-LLM       | -----> | Triton Inference     |
                             +--------------------+      +--------------------+
                                                 |                      |
                                                 |                      v
                                             +--------------------+
                                             | Model Repository    |
                                             +--------------------+
                                                 |                      |
                                                 |                      v
                                             +--------------------+
                                             | Backend Scheduler  |
                                             +--------------------+
                                                 |                      |
                                                 |                      v
                                             +--------------------+
                                             | Backend Providers  |
                                             +--------------------+
                                                 |                      |
                                                 |                      v
                                             +--------------------+
                                             |  HTTP/GRPC Server  |
                                             +--------------------+
```

**Explanation:**

* The diagram starts with **CUDA** at the bottom, representing the foundation for all the other tools.
* **TensorRT Parser** reads the pre-trained model.
* **TensorRT Optimizer** analyzes and optimizes the model for efficient execution on NVIDIA GPUs.
* **TensorRT Builder** creates a serialized **TensorRT Engine** optimized for the target GPU.
* **TensorRT Runtime** executes the optimized engine on the GPU for inference.
* **TensorRT-LLM** extends the functionalities of TensorRT for specifically optimizing and accelerating large language models.
* **Triton Inference Server** manages models in its **Model Repository**, uses the **Backend Scheduler** to distribute inference requests, and interacts with **Backend Providers** that can leverage optimized **TensorRT Engines** (including those from TensorRT-LLM) for GPU execution. Finally, it provides an **HTTP/GRPC Server** for clients to submit inference requests.

I hope this visual representation clarifies the relationships and architecture of these tools within NVIDIA's inference ecosystem.