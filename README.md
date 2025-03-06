# **Customized vSphere Machine Template for Dual NIC Deployment in TKGM 2.5.0**

## **Overview**
This repository contains a customized **vSphereMachineTemplate** with dual NIC support and a **ClusterClass** definition that integrates this template into a TKGM (Tanzu Kubernetes Grid Management) cluster. These configurations are intended for deploying worker nodes with two network interfaces—one for the primary network and another for storage or backend connectivity.

The guide below explains how the **VSphereMachineTemplate** and **ClusterClass** work together and how to apply them to an existing TKGM 2.5.0 cluster.

---

## **Files in This Repository**
- `vspheremachine-template-dual-nic.yaml`: Defines the VM template for worker nodes with dual NICs.
- `vsphereclusterclass-dual-nic.yaml`: Specifies the **ClusterClass** configuration that integrates the dual-NIC machine template.

---

## **How the vSphere Machine Template Works**
The `vspheremachine-template-dual-nic.yaml` file provides a worker node VM template with the following characteristics:

### **Key Configuration Settings**
- **Clone Mode:** `fullClone` (Ensures an independent copy for each machine)
- **Datacenter:** `<DATACENTER-NAME>` (Replace with your datacenter name)
- **Datastore:** `<DATASTORE-NAME>` (Specify the datastore for VM storage)
- **Disk Size:** `40GiB`
- **Memory:** `8GiB`
- **CPU:** `4` vCPUs
- **Networking:** **Two network interfaces**
  - First NIC: `<ORIGINAL-NETWORK>` (Primary network)
  - Second NIC: `<STORAGE-NETWORK>` (Storage or backend network)
- **vSphere Resource Details:**
  - **Folder:** `<FOLDER-NAME>`
  - **Resource Pool:** `<RESOURCE-NAME>`
  - **vCenter Server:** `<VCENTERSERVER-NAME>`

### **Networking Details**
This template provisions worker nodes with **two network interfaces**, allowing network segmentation between compute and storage traffic.

---

## **How the Cluster Class Works**
The `vsphereclusterclass-dual-nic.yaml` file defines a **ClusterClass** that includes a worker node deployment using the **dual NIC vSphereMachineTemplate**.

### **Key Configuration Features**
- **Control Plane Configuration:**
  - Machine health checks enabled
  - Uses a separate infrastructure template
- **Worker Node Configuration:**
  - Includes a deployment using the **tkg-vsphere-default-v1.2.0-worker-dual-nic** machine template
  - Supports both Linux and Windows worker node classes
- **Patch Management:**
  - External patching extension included

### **Worker Node Integration**
The worker node deployment references the **dual NIC vSphere machine template** as follows:

```yaml
workers:
  machineDeployments:
  - class: tkg-worker
    template:
      infrastructure:
        ref:
          apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
          kind: VSphereMachineTemplate
          name: tkg-vsphere-default-v1.2.0-worker-dual-nic
          namespace: default
```

---

## **How to Apply These Configurations to an Existing TKGM 2.5.0 Cluster**

### **Prerequisites**
1. **Ensure your environment meets the following requirements:**
   - A running TKGM 2.5.0 cluster
   - Access to vSphere with necessary permissions
   - Proper networking setup for dual NICs
   - `kubectl` configured to communicate with the TKGM cluster

2. **Verify that vSphere supports dual NIC configurations.**
   - Ensure the required VLANs or port groups exist in vSphere.
   - Confirm that the vSphere template is configured correctly.

---

### **Step-by-Step Deployment Guide**

#### **Step 1: Apply the vSphere Machine Template**
Modify the `vspheremachine-template-dual-nic.yaml` file with the correct **datacenter, datastore, network names, folder, resource pool, and vCenter details**, then apply it to the cluster.

```bash
kubectl apply -f vspheremachine-template-dual-nic.yaml
```

#### **Step 2: Apply the ClusterClass**
Apply the `vsphereclusterclass-dual-nic.yaml` file to integrate the machine template into the TKGM cluster.

```bash
kubectl apply -f vsphereclusterclass-dual-nic.yaml
```

#### **Step 3: Update Cluster to Use the New Class**
If you are using a managed cluster, update the cluster configuration to reference the new **ClusterClass**.

```yaml
apiVersion: cluster.x-k8s.io/v1beta1
kind: Cluster
metadata:
  name: my-cluster
  namespace: default
spec:
  topology:
    class: tkg-vsphere-default-v1.2.0
    version: v1.25.10+vmware.2-tkg.1
```

Apply the update:

```bash
kubectl apply -f my-cluster.yaml
```

#### **Step 4: Verify Deployment**
Check if the worker nodes are using the correct **dual NIC configuration**:

```bash
kubectl get nodes -o wide
```

Inspect the node details:

```bash
kubectl describe node <NODE-NAME>
```

Ensure the network interfaces are assigned correctly.

