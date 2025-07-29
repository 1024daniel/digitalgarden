---
{"dg-publish":true,"permalink":"/AI/ascend/测试记录/qwen3-235b w8a8/","noteIcon":"3"}
---

#ascend #tune
### 2.1.T10.B060

2k/2k输入卡3s, 100ms数据

TTFT 779ms TPOT 109.9ms

![](file:////Users/daniel/Library/Containers/com.kingsoft.wpsoffice.mac/Data/tmp/wps-daniel/ksohtml//wps1.jpg) 

### 2.1.RC1.B092版本

PD分离， 1P1D，P节点双机，D节点4机

user_config.json
```json
 {
  "version": "v1.0",
  "deploy_config": {
    "p_instances_num": 1,
    "d_instances_num": 1,
    "single_p_instance_pod_num": 2,
    "single_d_instance_pod_num": 4,
    "p_pod_npu_num": 8,
    "d_pod_npu_num": 8,
    "p_instances_scale_num": 0,
    "d_instances_scale_num": 0,
    "model_id": "",
    "prefill_distribute_enable": 0,
    "decode_distribute_enable": 1,
    "image_name": "b092_qwen3:v0",
    "job_id": "mindie",
    "hardware_type": "800I_A2",
    "mindie_env_path": "./conf/mindie_env.json",
    "mindie_host_log_path": "/root/log/ascend_log",
    "mindie_container_log_path": "/root/mindie",
    "weight_mount_path": "/home/weight/Qwen3-235B-A22B-W8A8",
    "coordinator_backup_cfg": {
      "function_enable": false
    },
    "controller_backup_cfg": {
      "function_sw": false
    },
    "deploy_mount_path": {
      "ms_controller_mount": {
        "$user_define_host_path": "$user_define_container_path"
      },
      "ms_coordinator_mount": {
        "$user_define_host_path": "$user_define_container_path"
      },
      "prefill_server_mount": {
        "$user_define_host_path1": "$user_define_container_path",
        "$user_define_host_path2": "$user_define_container_path"
      },
      "decode_server_mount": {
        "$user_define_host_path1": "$user_define_container_path",
        "$user_define_host_path2": "$user_define_container_path"
      }
    },
    "tls_config": {
      "tls_enable": false,
      "kmc_ksf_master": "./security/master/tools/pmt/master/ksfa",
      "kmc_ksf_standby": "./security/standby/tools/pmt/standby/ksfb",
      "infer_tls_items": {
        "ca_cert": "./security/infer/security/certs/ca.pem",
        "tls_cert": "./security/infer/security/certs/cert.pem",
        "tls_key": "./security/infer/security/keys/cert.key.pem",
        "tls_passwd": "./security/infer/security/pass/key_pwd.txt",
        "tls_crl": ""
      },
      "management_tls_items": {
        "ca_cert": "./security/management/security/certs/management_ca.pem",
        "tls_cert": "./security/management/security/certs/cert.pem",
        "tls_key": "./security/management/security/keys/cert.key.pem",
        "tls_passwd": "./security/management/security/pass/key_pwd.txt",
        "tls_crl": ""
      },
      "cluster_tls_enable": false,
      "cluster_tls_items": {
        "ca_cert": "./security/clusterd/security/certs/ca.pem",
        "tls_cert": "./security/clusterd/security/certs/cert.pem",
        "tls_key": "./security/clusterd/security/keys/cert.key.pem",
        "tls_passwd": "./security/clusterd/security/pass/key_pwd.txt",
        "tls_crl": ""
      },
      "etcd_server_tls_enable": false,
      "etcd_server_tls_items": {
        "ca_cert": "./security/etcd_server/security/certs/ca.pem",
        "tls_cert": "./security/etcd_server/security/certs/cert.pem",
        "tls_key": "./security/etcd_server/security/keys/cert.key.pem",
        "tls_passwd": "./security/etcd_server/security/pass/key_pwd.txt",
        "kmc_ksf_master": "./security/etcd_server/tools/pmt/master/ksfa",
        "kmc_ksf_standby": "./security/etcd_server/tools/pmt/standby/ksfb",
        "tls_crl": ""
      }
    }
  },
  "mindie_ms_controller_config": {
    "deploy_mode": "pd_separate",
    "digs_prefill_slo": 1000,
    "digs_decode_slo": 50,
    "multi_node_infer_config": {
      "multi_node_infer_enable": true
    }
  },
  "mindie_ms_coordinator_config": {
    "http_config": {
      "predict_ip": "127.0.0.1",
      "predict_port": "1025",
      "manage_ip": "127.0.0.1",
      "manage_port": "1026",
      "server_thread_num": 10,
      "client_thread_num": 10,
      "http_timeout_seconds": 360,
      "keep_alive_seconds": 180
    },
    "request_limit": {
      "single_node_max_requests": 4096,
      "max_requests": 10000
    },
    "exception_config": {
      "first_token_timeout": 3600,
      "infer_timeout": 3600
    }
  },
  "mindie_server_prefill_config": {
    "ServerConfig": {
      "maxLinkNum": 4096,
      "inferMode": "dmi",
      "tokenTimeout": 3600,
      "e2eTimeout": 3600,
      "distDPServerEnabled": false
    },
    "BackendConfig": {
      "npuDeviceIds": [
        [
          0,
          1,
          2,
          3,
          4,
          5,
          6,
          7
        ]
      ],
      "tokenizerProcessNumber": 1,
      "multiNodesInferEnabled": true,
      "ModelDeployConfig": {
        "maxSeqLen": 8192,
        "maxInputTokenLen": 6144,
        "ModelConfig": [
          {
            "modelInstanceType": "Standard",
            "modelName": "qwen3",
            "modelWeightPath": "/home/weight/Qwen3-235B-A22B-W8A8",
            "worldSize": 8,
            "cpuMemSize": 5,
            "npuMemSize": -1,
            "backendType": "atb",
            "trustRemoteCode": false,
            "dp": 16,
            "cp": 1,
            "tp": 1,
            "sp": 1,
            "moe_ep": 4,
            "pp": 1,
            "moe_tp": 4,
            "kv_link_timeout": 1080,
            "modelCutPolicy": "custom",
            "models": {
              "deepseekv2": {
                "kv_cache_options": {
                  "enable_nz": true
                },
                "ep_level": 1,
                "enable_init_routing_cutoff": false,
                "topk_scaling_factor": 1.0
              }
            }
          }
        ]
      },
      "ScheduleConfig": {
        "maxPrefillBatchSize": 16,
        "maxPrefillTokens": 6144
      }
    }
  },
  "mindie_server_decode_config": {
    "ServerConfig": {
      "maxLinkNum": 256,
      "fullTextEnabled": false,
      "inferMode": "dmi",
      "tokenTimeout": 3600,
      "e2eTimeout": 3600,
      "distDPServerEnabled": true
    },
    "BackendConfig": {
      "npuDeviceIds": [
        [
          0
        ]
      ],
      "tokenizerProcessNumber": 1,
      "multiNodesInferEnabled": false,
      "ModelDeployConfig": {
        "maxSeqLen": 8192,
        "maxInputTokenLen": 6144,
        "truncation": false,
        "ModelConfig": [
          {
            "modelInstanceType": "Standard",
            "modelName": "qwen3",
            "modelWeightPath": "/home/weight/Qwen3-235B-A22B-W8A8",
            "worldSize": 1,
            "cpuMemSize": 5,
            "npuMemSize": -1,
            "backendType": "atb",
            "trustRemoteCode": false,
            "dp": 32,
            "cp": 1,
            "tp": 1,
            "sp": 1,
            "moe_ep": 32,
            "pp": 1,
            "moe_tp": 1,
            "kv_trans_timeout": 10,
            "kv_link_timeout": 1080,
            "modelCutPolicy": "custom",
            "models": {
              "deepseekv2": {
                "kv_cache_options": {
                  "enable_nz": true
                }
              }
            }
          }
        ]
      },
      "ScheduleConfig": {
        "distributedEnable": true,
        "maxPrefillBatchSize": 2,
        "maxPrefillTokens": 6144,
        "maxBatchSize": 64,
        "maxIterTimes": 2048,
        "maxQueueDelayMicroseconds": 5000
      }
    }
  }
}

```



mindie_env.json：
增加两行:
`"HCCL_INTRA_PCIE_ENABLE": 1,`
`"HCCL_INTRA_ROCE_ENABLE": 0`
删除`"ATB_LLM_ENABLE_AUTO_TRANSPOSE": 0,`


```json
{
  "version": "1.0.0",
  "mindie_common_env": {
    "CANN_INSTALL_PATH": "/usr/local/Ascend",
    "MIES_INSTALL_PATH": "/usr/local/Ascend/mindie/latest/mindie-service",
    "MINDIE_LLM_HOME_PATH": "/usr/local/Ascend/mindie/latest/mindie-llm",
    "TASK_QUEUE_ENABLE": 1,
    "HCCL_BUFFSIZE": 120,
    "MINDIE_LOG_TO_FILE": 1,
    "MINDIE_LOG_TO_STDOUT": 1,
    "MINDIE_LOG_LEVEL": "INFO",
    "ASCEND_SLOG_PRINT_TO_STDOUT": 0,
    "ASCEND_GLOBAL_LOG_LEVEL": 3,
    "ASCEND_GLOBAL_EVENT_ENABLE": 1,
    "ATB_LOG_TO_FILE": 0,
    "ASDOPS_LOG_TO_STDOUT": 0,
    "ASDOPS_LOG_LEVEL": "ERROR",
    "MINDIE_LLM_CONTINUOUS_BATCHING": 1,
    "MINDIE_LLM_RECOMPUTE_THRESHOLD": 0.5,
    "DIST_PD_DISAGGREGATION": 1,
    "HCCL_RDMA_RETRY_CNT": 7,
    "HCCL_RDMA_TIMEOUT": 18,
    "HCCL_EXEC_TIMEOUT": 60
  },
  "mindie_ms_controller_env": {
  },
  "mindie_ms_coordinator_env": {
  },
  "mindie_server_prefill_env": {
    "PYTORCH_NPU_ALLOC_CONF": "expandable_segments:True",
    "OMP_NUM_THREADS": 8,
    "HCCL_CONNECT_TIMEOUT": 7200,
    "HCCL_ENTRY_LOG_ENABLE": 1,
    "HCCL_ALGO": "level0:NA;level1:pipeline",
    "DP_MOVE_UP_ENABLE": 1,
    "MINDIE_ASYNC_SCHEDULING_ENABLE": 0,
    "HCCL_OP_EXPANSION_MODE": "AIV",
    "NPU_MEMORY_FRACTION": 0.92
  },
  "mindie_server_decode_env": {
    "MINDIE_ENABLE_DP_DISTRIBUTED": 1,
    "DP_PARTITION_UP_ENABLE": 1,
    "DP_MOVE_UP_ENABLE": 1,
    "HCCL_BUFFSIZE": 512,
    "ATB_LAYER_INTERNAL_TENSOR_REUSE": 1,
    "ATB_CONVERT_NCHW_TO_ND": 1,
    "ATB_CONTEXT_WORKSPACE_SIZE": 0,
    "ATB_LAUNCH_KERNEL_WITH_TILING": 1,
    "ATB_LLM_HCCL_ENABLE": 1,
    "TASK_QUEUE_ENABLE": 1,
    "INF_NAN_MODE_ENABLE": 0,
    "PYTORCH_NPU_ALLOC_CONF": "expandable_segments:True",
    "OMP_NUM_THREADS": 8,
    "NPU_MEMORY_FRACTION": 0.92,
    "HCCL_ENTRY_LOG_ENABLE": 1,
    "HCCL_CONNECT_TIMEOUT": 7200,
    "MINDIE_ASYNC_SCHEDULING_ENABLE": 1,
    "HCCL_INTRA_PCIE_ENABLE": 1,
    "HCCL_INTRA_ROCE_ENABLE": 0
  }
}

```

不设置rr先拉大并发将tpot打满:
![Pasted image 20250715105209.png](/img/user/AI/ascend/%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95/attachments/Pasted%20image%2020250715105209.png)
设置rr尝试降低TTFT，发现Median TTFT已经超过要求的3s

![Pasted image 20250715105247.png](/img/user/AI/ascend/%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95/attachments/Pasted%20image%2020250715105247.png)

设置输出长度为2，测试系统qps,qps为7
![Pasted image 20250715105716.png](/img/user/AI/ascend/%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95/attachments/Pasted%20image%2020250715105716.png)
单并发测试TTFT最小值，发现也略超3s，高并发肯定会超过3s
![Pasted image 20250715105600.png](/img/user/AI/ascend/%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95/attachments/Pasted%20image%2020250715105600.png)

对齐推荐的rr测试，TTFT接近5s
![Pasted image 20250715105916.png](/img/user/AI/ascend/%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95/attachments/Pasted%20image%2020250715105916.png)

参数优化
![Pasted image 20250715110303.png](/img/user/AI/ascend/%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95/attachments/Pasted%20image%2020250715110303.png)
boot.sh：

![Pasted image 20250715110207.png](/img/user/AI/ascend/%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95/attachments/Pasted%20image%2020250715110207.png)


测试结果:


![Pasted image 20250715110420.png](/img/user/AI/ascend/%E6%B5%8B%E8%AF%95%E8%AE%B0%E5%BD%95/attachments/Pasted%20image%2020250715110420.png)

