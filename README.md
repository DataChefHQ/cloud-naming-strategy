# **Hybrid Domain-Driven Resource Naming & Tagging Strategy for AWS, GCP and Azure**

## Version History

| Version | Date       | Description                                                                              |
|---------|------------|------------------------------------------------------------------------------------------|
| 1.0.0   | 2025-02-19 | Initial public release of the hybrid domain-driven naming and tagging strategy document. |

## **1. Introduction**

Without a clear naming and tagging strategy, cloud environments become
chaotic, making it hard to track costs, enforce security, and manage
resources. Inconsistent naming leads to confusion, increases
operational overhead, and slows down troubleshooting. Poor tagging
causes visibility issues, making automation, compliance, and reporting
unreliable.

When working across **AWS**, **Azure**, and **GCP**, a consistent and
**domain-focused** naming and tagging strategy helps:

1. **Identify** resource owners and intended purposes.
2. **Track costs** per team or product.
3. **Apply governance and compliance** (e.g., data classification).
4. **Automate** certain operational tasks (backups, lifecycle
   management, etc.).

This document defines a **short, domain-driven naming convention**
plus a **robust tagging framework**, ensuring clarity, scalability,
and alignment with **Well-Architected** best practices.

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [**Hybrid Domain-Driven Resource Naming & Tagging Strategy**](#hybrid-domain-driven-resource-naming--tagging-strategy)
  - [**1. Introduction**](#1-introduction)
  - [**2. Why a Domain-Driven Approach?**](#2-why-a-domain-driven-approach)
  - [**3. Naming Convention**](#3-naming-convention)
    - [**3.1 Short, Domain-Centric Pattern**](#31-short-domain-centric-pattern)
    - [**3.2 Cloud-Specific Adjustments**](#32-cloud-specific-adjustments)
    - [Tips](#tips)
  - [**4. Tagging / Labeling Strategy**](#4-tagging--labeling-strategy)
    - [**4.1 Recommended Tag/Label Keys**](#41-recommended-taglabel-keys)
    - [**4.2 Policy Enforcement**](#42-policy-enforcement)
  - [**5. Operationalizing the Strategy**](#5-operationalizing-the-strategy)
    - [**5.1 Infrastructure as Code (IaC)**](#51-infrastructure-as-code-iac)
    - [**5.2 Governance & Compliance**](#52-governance--compliance)
    - [**5.3 Education & Adoption**](#53-education--adoption)
  - [**6. Example End-to-End Scenario**](#6-example-end-to-end-scenario)
    - [**6.1 Use Case**](#61-use-case)
    - [**6.2 Tagging / Labeling**](#62-tagging--labeling)
  - [**7. Alignment with AWS and Azure Well-Architected**](#7-alignment-with-aws-and-azure-well-architected)
  - [**8. Conclusion**](#8-conclusion)
    - [**Key Advantages**](#key-advantages)

<!-- markdown-toc end -->

---

## **2. Why a Domain-Driven Approach?**

- **Clarity**: Embeds your business domains (e.g., Marketing, Finance)
  or key applications (e.g., “Analytics”, “Billing”) in resource
  names, making it obvious which function the resource supports.
- **Ownership**: Easier for teams to see which domain or application
  they’re responsible for.
- **Streamlined Governance**: Domain-driven naming aligns with your
  organizational structure, so policies, cost allocation, and security
  practices map naturally to your business capabilities.

---

## **3. Naming Convention**

### **3.1 Short, Domain-Centric Pattern**

Cloud resources all have different length and character
requirements. Instead of packing *every* detail (like team, cost
center) into the resource name, store additional meta data in **tags**
(AWS/Azure) or **labels** (GCP).

This approach **requires** `domain/project`, `environment` to be
defined with the resource name. Optionally `region`, `resource type`
and `optional id` can be used to further group resources, just keep in
consideration the name’s maximum length.

**Naming Pattern**:

```
<DomainOrProject>-<Environment>-<ResourceName>
<DomainOrProject>-<Environment>-<Region>-<ResourceType>-<ResourceName>-<UniqueID>
```

1. **DomainOrProject**
    - The primary domain or application name.
    - Examples: `marketing`, `finance`.
2. **Environment**
    - Standard designations such as `dev`, `test`, `uat`, `prod`.
3. **Region** *(Optional)*
    - Short code for the cloud region, e.g., `use1` = `us-east-1`.
    - Use consistent abbreviations across clouds.
4. **ResourceType** *(Optional)*
    - Short 2–4 letter code describing the resource: `db`, `vm`,
      `stor`, `func`, etc.
5. **UniqueID** *(Optional)*
    - Short numeric or alphanumeric identifier if global uniqueness is
      required. Examples: `001`, `abc1`.
6. **ResourceName**
    - The name for you're resource

**Example**:

```
projecta-prod-masterdb
marketing-dev-use1-db-masterdb-01
```

### **3.2 Cloud-Specific Adjustments**

<aside> ☝

### Tips

- Keep all naming parts as short as possible, by using abbreviations
  where possible, for example; `mktg` instead of`marketing` and `prod`
  instead of `production`.
- Remove white space within a name part, so instead of `master-db`
write `masterdb`. This is because the `-` symbol is used in our
consistent naming format.  </aside>

Not all resources follow a standard naming convention; they vary in
length, allowed characters, and formatting rules. Here are a few such
edge cases:

- **AWS**:
    - For S3 buckets, must be lowercase letters, numbers, hyphens only
      (no underscores).
    - EC2, RDS, and other services typically allow hyphens,
      alphanumeric, and sometimes underscores.
- **Azure**:
    - **Storage accounts**: 3–24 characters, all lowercase
      letters/numbers. You may need to abbreviate.
    - **Resource groups**: 1–90 characters, can use alphanumeric,
      underscores, parentheses, hyphens, periods.
- **GCP**:
    - **Project IDs**: 6–30 characters, must start with a letter, only
      lowercase letters, digits, and hyphens.
    - **Compute Engine**: Typically lowercase letters, digits, and
      hyphens, up to 63 chars.

---

## **4. Tagging / Labeling Strategy**

**Tags** (AWS & Azure) and **Labels** (GCP) let you store additional
metadata that doesn’t need to appear in the resource name.

### **4.1 Recommended Tag/Label Keys**

**ESSENTIALS**

| **Key** | **Purpose** | **Example Values** |
|--------------------|-------------------------------------------|------------------------------|
| **Domain/Project** | High-level business area or project name. | `marketing`, `projecta` |
| **Owner** | Contact email or distribution list.  | `data-eng@company.com` |
| **Environment** | Environment name.  | `dev`, `test`, `uat`, `prod` |

**OPTIONAL**

| **Key** | **Purpose** | **Example Values** |
|------------------------|--------------------------------------------------------------|----------------------------------------|
| **Name** | The resource’s friendly name, mirroring your naming pattern. | `marketing-dev-use1-db-01` |
| **TeamOrBU** | Team or business unit responsible.  | `DataEng`, `AppDev`, `FinOps` |
| **CostCenter** | Finance/billing reference.  | `CC1234`, `BU5678` |
| **DataClassification** | Indicates sensitivity (e.g., `internal`, `public`, `PII`).  | `internal`, `public`, `pii` |
| **Compliance** | Regulatory tags (e.g., `GDPR`, `HIPAA`, `ISO27001`).  | `GDPR`, `SOX` |
| **CreatedBy** | Identifies provisioning mechanism or user.  | `terraform`, `ci-pipeline`, `john.doe` |
| **Expiration** | Date or condition for decommissioning if ephemeral.  | `2025-12-31` |

### **4.2 Policy Enforcement**

<aside> ‼️

You can not be assured that all of your resources will be tagged if
implementing only enforcement. The enforcement strategies listed below
do not support 100% of that cloud’s resources. See 5.2 for operational
audits.

</aside>

- **AWS**: Requires a range of services for an effective strategy;
  Organization SCPs, Organization Tag Policies and AWS Config rules
- **Azure**: Use **Azure Policy** to deny, append, or modify tags at
  resource creation.
- **GCP**: Use **Organization Policies** or custom Cloud Functions to
  check and correct labels.

---

## **5. Operationalizing the Strategy**

### **5.1 Infrastructure as Code (IaC)**

1. **Common Modules**:
    - Create Terraform/Bicep/CloudFormation(CDK) modules that accept
      inputs like `<DomainOrProject>`, `<Environment>`, `<Region>`,
      `<ResourceType>` and generate a **standardized name** plus
      **mandatory tags**.
2. **Pipeline Enforcement**:
    - Include naming/tagging checks in your CI/CD pipeline. A pipeline
      step can verify that each resource block follows the pattern and
      required tag keys. Use tools like checkov, CFN nag (or CDK nag)
      to enforce this at build time.
3. **Templates**:
    - Provide templates or example code for typical resource types
      (e.g., VM, container registry, database, storage) so teams can
      replicate the pattern quickly.

### **5.2 Governance & Compliance**

- **Regular Audits**:
    - Scan resources to ensure all are properly named and
      tagged. Tools like **AWS Config**, **Azure Resource Graph**,
      **GCP Asset Inventory** can assist.
- **Automated Remediation**:
    - Optionally configure serverless functions (AWS Lambda, Azure
      Functions, GCP Cloud Functions) to fix or notify owners about
      non-compliant resources.

### **5.3 Education & Adoption**

- **Style Guides**:
    - Provide a single reference doc with standard abbreviations for
      domains, resource types, and region codes.
- **Workshops**:
    - Run short training sessions explaining the importance of
      domain-driven naming, how to use the modules, and how to pass
      the correct variables in IaC.
- **Quick Reference Examples**:
    - Show examples for each resource type in each environment. Make
      sure new hires and existing teams can find these quickly.

---

## **6. Example End-to-End Scenario**

### **6.1 Use Case**

- **Domain/Project**: `marketing`
- **Environment**: `prod`
- **Region**: `use1` (AWS US East 1)
- **ResourceType**: `db` (database)
- **ResourceName:** `masterdb`
- **UniqueID**: `001`

**Resource Name**:

```
marketing-prod-use1-db-masterdb-001
```

### **6.2 Tagging / Labeling**

**Required Tags** (AWS) or **Labels** (GCP) example:

- `Name = "Billing-prod-use1-db-001"`
- `DomainOrProject = "Finance"` (if “Billing” is a subdomain of
  Finance)
- `TeamOrBU = "FinOps"`
- `Environment = "prod"`
- `Owner = "finops-team@company.com"`
- `CostCenter = "CC9876"`
- `DataClassification = "internal"`
- `Compliance = "SOX"`
- `CreatedBy = "terraform"`
- `Expiration = ""` (not applicable if permanent)

By following this approach:

- The **resource name** is concise yet descriptive.
- **Tags** hold details about domain, ownership, cost center, data
  classification, etc.

---

## **7. Alignment with AWS and Azure Well-Architected**

Both AWS and Azure Well-Architected frameworks emphasize:

1. **Consistent Naming**: Simplifies operational tasks, resource
   discovery, and automation.
2. **Detailed Tagging**: Supports cost optimization, security,
   compliance, and operational excellence pillars.

This hybrid strategy:

- Keeps the resource **name** short, focusing on environment, region,
  resource type, and the broader domain/application.
- Leverages **tags**/labels for ownership, cost, compliance, and data
  classification—directly aligning with recommended practices for cost
  allocation, security posture, and operational governance.

---

## **8. Conclusion**

### **Key Advantages**

- **Simplicity**: Short, domain-driven names are easy to read,
  remember, and type.
- **Clarity**: Teams instantly see which environment (e.g., `prod`)
  and region (e.g., `use1`) a resource belongs to.
- **Robust Metadata**: Detailed tagging satisfies governance, cost
  tracking, and compliance requirements without cluttering the
  resource name.
- **Cross-Cloud Consistency**: Works equally well in **AWS**,
  **Azure**, and **GCP** with minimal adjustments for each platform’s
  naming limits.

By **combining** a concise resource naming pattern with
**comprehensive tagging**, you strike a balance between **immediate
clarity** and **rich metadata**—exactly what data and application
teams need to streamline operations, control costs, and maintain
compliance in a multi-cloud environment.
