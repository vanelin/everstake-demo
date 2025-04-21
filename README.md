# Legend

### SaaS Omni orchestrates lifecycle of all Talos nodes.

- KubeSpan creates a WireGuard mesh so every control‑plane and worker node
  sees a single, private /24 overlay—no VPNs or flat networks required.

- Control‑plane (3 × Talos masters on AWS EC2):
    - Runs etcd quorum, kube‑api‑server, controller‑manager, scheduler.
    - Exposed only on WireGuard IPs; no public 6443.
    - Hosts a 3‑pod HashiCorp Vault cluster in integrated storage (raft)
      mode—one pod per master for HA.

- Worker pool (bare‑metal Talos servers):
    - Hosts stateful Ethereum & Tron blockchain pods (1 replica each; data
      on local NVMe).


- All intra‑cluster traffic—including etcd, Vault gossip, and blockchain
  peer‑to‑peer sync—flows through the encrypted KubeSpan overlay.

```mermaid
flowchart TD
    %% Top level - Omni Orchestrator
    omni["SaaS Omni Orchestrator
    (omni.siderolabs.com - cluster manager)"]
    
    %% API Connection
    omni -->|"Talos API (mTLS)"| kubespan
    
    %% KubeSpan layer
    subgraph kubespan["KubeSpan Overlay (WireGuard) - end-to-end encryption"]
    end
    
    %% Cloud environments
    kubespan --> aws
    kubespan --> baremetal
    
    %% AWS Cloud section
    subgraph aws["AWS Cloud
    (us-east-1)"]
        subgraph controlplane["+Control-Plane ASG"]
            master1["master-1 (EC2 AZ-a)
            • kube-api-server
            • Vault HA (quorum)"]
            master2["master-2 (EC2 AZ-b)
            • kube-api-server
            • Vault HA (quorum)"]
            master3["master-3 (EC2 AZ-c)
            • kube-api-server
            • Vault HA (quorum)"]
        end
    end
    
    %% Bare-metal section
    subgraph baremetal["Bare-Metal DC
    (On-prem / Colo rack)"]
        subgraph workers["+Worker Pool"]
            worker1["worker-1 (Talos ISO)
            • Solana-node pod"]
            worker2["worker-2 (Talos ISO)
            • Ethereum-node pod"]
            worker3["worker-3 (Talos ISO)
            • Tron-node pod"]
        end
    end
    
    %% Connect control-plane to worker nodes with etcd quorum
    controlplane <-->|"etcd quorum
    (encrypted via KubeSpan)"| workers
    
    %% Styling
    classDef omni fill:#9370DB,stroke:#4B0082,stroke-width:2px,color:white
    classDef kubespan fill:#777777,stroke:#333333,stroke-width:2px,color:white
    classDef aws fill:#FF9900,stroke:#232F3E,stroke-width:2px,color:black
    classDef baremetal fill:#3970E4,stroke:#0A2272,stroke-width:2px,color:white
    
    class omni omni
    class kubespan kubespan
    class aws,controlplane,master1,master2,master3,aws_api aws
    class baremetal,workers,worker1,worker2,worker3 baremetal
```