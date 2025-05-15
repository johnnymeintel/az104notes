# Implement and Manage Storage (15-20%)

### Objective Overview

Refer to individual objective pages for details:

- [[01 - Configure access to storage]]
- [[02 - Configure and manage storage accounts]]
- [[03 - Configure Azure Files and Azure Blob Storage]]

---

### Key Concepts

- Storage account as container for all Azure Storage services
- Difference between management plane (ARM) and data plane access
- Storage tiers optimize cost based on access patterns
- Data redundancy protects against failures at different scales
- Identity-based access vs key-based access

### Relevant Terms & Definitions

|Term|Definition|Portal Location|
|---|---|---|
|Storage Account|Top-level Azure resource containing blobs, files, queues, tables|Home > Storage accounts|
|Container|Logical grouping for blobs within a storage account|Storage account > Containers|
|File Share|SMB protocol-based file storage|Storage account > File shares|
|SAS Token|URI that grants restricted access rights to Azure Storage resources|Storage account > Shared access signature|
|Access Tier|Performance/cost level for blob storage (Hot/Cool/Archive)|Blob > Change tier|

### Service Comparison

|Feature|Azure Blob Storage|Azure Files|Azure Disks|
|---|---|---|---|
|Protocol|REST API|SMB/NFS|Block storage|
|Use Case|Unstructured data|Lift-and-shift apps|VM disks|
|Max Size|5PB per account|100TB per share|64TB per disk|

---

### Azure CLI Commands

```bash
# Create storage account
az storage account create --name mystorageaccount --resource-group MyRG --location eastus --sku Standard_LRS

# Generate SAS token
az storage account generate-sas --account-name mystorageaccount --permissions rwdlac --expiry 2024-12-31

# Create blob container
az storage container create --account-name mystorageaccount --name mycontainer

# Set blob tier
az storage blob set-tier --account-name mystorageaccount --container-name mycontainer --name myblob --tier Cool
```

### PowerShell Commands

```powershell
# Create storage account
New-AzStorageAccount -ResourceGroupName "MyRG" -Name "mystorageaccount" -Location "eastus" -SkuName "Standard_LRS"

# Get storage account key
Get-AzStorageAccountKey -ResourceGroupName "MyRG" -Name "mystorageaccount"

# Create file share
New-AzStorageShare -Name "myshare" -Context $ctx

# Enable soft delete
Enable-AzStorageBlobDeleteRetentionPolicy -ResourceGroupName "MyRG" -AccountName "mystorageaccount" -RetentionDays 7
```

---

### Cost Considerations

- Storage capacity (per GB)
- Number of transactions/operations
- Data egress charges (outbound transfer)
- Redundancy type affects pricing (LRS cheapest, GZRS most expensive)
- Archive tier cheapest for storage but expensive for retrieval

### Security Best Practices

- Enable firewall rules and restrict public access
- Use Azure AD authentication over access keys when possible
- Rotate access keys regularly
- Implement least-privilege access with RBAC
- Enable Advanced Threat Protection for storage accounts

### Business Use Cases

- Media storage and streaming (Blob storage with CDN)
- File server replacement (Azure Files)
- Backup and disaster recovery (GRS/GZRS)
- Data archival for compliance (Archive tier)
- Application data and logs (Hot/Cool tiers)

### Real-World Analogies

- Storage account = Office building with different departments
- Containers = File cabinets within departments
- Access tiers = Different types of storage rooms (active files vs archive basement)
- SAS tokens = Temporary visitor passes with specific permissions
- Redundancy = Backup copies in different locations

### Common Pitfalls & Misconceptions

- Not understanding early deletion penalties for Cool/Archive tiers
- Confusing storage account types (StorageV2 vs BlobStorage)
- Assuming all services support all authentication methods
- Forgetting that archive tier requires rehydration before access
- Not considering data egress costs in multi-region scenarios

### Things to Look Out for on the Exam

- Know the differences between redundancy options (LRS, ZRS, GRS, GZRS)
- Understand SAS token types and when to use each
- Remember tier transition restrictions (Hot->Cool->Archive only)
- Be familiar with retention period limits for soft delete
- Know which features require specific storage account types


---

#### Practice Test Questions I Got Wrong

##### Question 1

**Question:**

**Options:**

- A.
- B.
- C.
- D.

**Correct Answer:**

**Why I Got It Wrong:**

**What to Remember:**


---

##### Question 2

**Question:**

**Options:**

- A.
- B.
- C.
- D.

**Correct Answer:**

**Why I Got It Wrong:**

**What to Remember:**


---

#### Practice Scenarios

##### Scenario 1:



---

#### Project Ideas

- 