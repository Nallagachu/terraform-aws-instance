
# Terraform AWS Instance Module

[](https://www.google.com/search?q=LICENSE)
[](https://releases.hashicorp.com/terraform/)
[](https://aws.amazon.com/)

Welcome to the `terraform-aws-instance` repository\! This project provides a robust and reusable Terraform module designed to provision an AWS EC2 instance along with its essential security group. It's crafted to exemplify core Infrastructure as Code (IaC) best practices, including modularity, controlled variable inputs, and secure remote state management.

Whether you're just moving beyond basic resource definitions or looking for a clean, practical example of a reusable Terraform module, this repository serves as an excellent reference.

## Table of Contents

1.  [Project Overview](https://www.google.com/search?q=%231-project-overview)
2.  [Key Features & Benefits](https://www.google.com/search?q=%232-key-features--benefits)
3.  [Module Architecture](https://www.google.com/search?q=%233-module-architecture)
4.  [Prerequisites](https://www.google.com/search?q=%234-prerequisites)
5.  [Getting Started](https://www.google.com/search?q=%235-getting-started)
      * [1. Clone the Repository](https://www.google.com/search?q=%231-clone-the-repository)
      * [2. Configure AWS S3 Backend](https://www.google.com/search?q=%232-configure-aws-s3-backend)
      * [3. Initialize Terraform](https://www.google.com/search?q=%233-initialize-terraform)
      * [4. Plan & Apply Your Infrastructure](https://www.google.com/search?q=%234-plan--apply-your-infrastructure)
6.  [Module Inputs](https://www.google.com/search?q=%236-module-inputs)
7.  [Module Outputs](https://www.google.com/search?q=%237-module-outputs)
8.  [Cleaning Up Resources](https://www.google.com/search?q=%238-cleaning-up-resources)
9.  [⚡ Important Note on State Locking ⚡](https://www.google.com/search?q=%239-important-note-on-state-locking)
10. [Future Scope & Learning Journey](https://www.google.com/search?q=%2310-future-scope--learning-journey)
11. [Contributing](https://www.google.com/search?q=%2311-contributing)
12. [License](https://www.google.com/search?q=%2312-license)
13. [Contact](https://www.google.com/search?q=%2313-contact)

## 1\. Project Overview

This project marks a significant step in understanding advanced Terraform concepts. Moving beyond simply defining individual resources, we've encapsulated the creation of a secure AWS EC2 instance into a **reusable Terraform module**. This module demonstrates:

  * **True Modularity:** How to create self-contained, shareable infrastructure components.
  * **Controlled Provisioning:** Implementing governance by defining and restricting the types of instances users can deploy, ensuring compliance and cost efficiency.
  * **Secure & Collaborative State Management:** Utilizing an encrypted Amazon S3 bucket for remote Terraform state, crucial for team environments and disaster recovery.
  * **Transparent Infrastructure:** Providing clear outputs to easily understand and integrate with the deployed resources.

This setup is ideal for teams or individuals seeking a consistent, maintainable, and securely managed approach to deploying basic compute infrastructure on AWS.

## 2\. Key Features & Benefits

  * **Reusable & Consistent Deployments:** Call the module multiple times across different environments or projects, ensuring uniformity in your EC2 deployments.
  * **Centralized Control & Maintenance:** Logic for instance and security group creation lives in one place. Updates or fixes within the module automatically benefit all consuming configurations, drastically reducing maintenance overhead.
  * **Policy-Driven Provisioning:** Variables are designed with predefined allowed values (e.g., `instance_type`), guiding users towards approved, cost-effective choices and preventing unintended resource sprawl.
  * **Robust State Management:** Your `tfstate` file is stored remotely in an **encrypted S3 bucket**, providing:
      * **Security:** Protection of sensitive state data at rest.
      * **Durability:** Safeguarding your state from local machine failures.
      * **Version History:** S3 bucket versioning keeps a complete audit trail of state changes, enabling effortless rollbacks.
  * **Clear Visibility:** Module outputs offer immediate insight into the provisioned resources (e.g., public IP, instance ID), simplifying integration and verification steps.
  * **Reduced Boilerplate:** Abstract common infrastructure patterns into this module, minimizing repetitive code in your main configurations.

## 3\. Module Architecture

The project is thoughtfully structured to clearly separate the module's definition from its consumption, promoting best practices in code organization:

```
.
├── main.tf                     # Main configuration: calls the 'ec2-instance' module, defines AWS provider, and S3 remote backend
├── variables.tf                # Defines input variables for the root module (e.g., global 'region')
├── outputs.tf                  # Defines output values from the root module for user convenience
├── versions.tf                 # Specifies required Terraform and AWS provider versions
├── .gitignore                  # Standard Git ignore file for Terraform artifacts (.terraform/, *.tfstate)
└── modules/
    └── ec2-instance/           # ✨ The reusable AWS EC2 Instance Module ✨
        ├── main.tf             # Defines the aws_instance and aws_security_group resources within the module
        ├── variables.tf        # Defines input variables specifically for THIS module (e.g., 'instance_type', 'ami_id')
        ├── outputs.tf          # Defines output values specifically from THIS module (e.g., 'public_ip', 'security_group_id')
        └── versions.tf         # Specifies required provider versions specific to the module
```

## 4\. Prerequisites

To get this project running in your AWS account, ensure you have the following installed and configured:

  * **Terraform CLI:** Install [Terraform v1.x.x or higher](https://developer.hashicorp.com/terraform/downloads).
  * **AWS CLI:** Configure the [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) with credentials that possess the necessary permissions to:
      * Create and manage EC2 instances and Security Groups.
      * Create and manage S3 buckets (for remote state storage).
      * *(Highly Recommended)* Permissions to enable encryption on your S3 bucket.

## 5\. Getting Started

Follow these steps to quickly deploy an EC2 instance using this module.

### 1\. Clone the Repository

Begin by cloning this repository to your local machine:

```bash
git clone https://github.com/Nallagachu/terraform-aws-instance.git
cd terraform-aws-instance
```

### 2\. Configure AWS S3 Backend

Before executing any Terraform commands, you **must create a dedicated S3 bucket** in your AWS account. This bucket will serve as the secure remote backend for your Terraform state file.

  * **Choose a Globally Unique Bucket Name:** (e.g., `your-organization-tf-states-project-12345`).
  * **Enable Bucket Versioning:** This is critical for maintaining a historical record of your state file, allowing for easy rollbacks if needed.

**Example AWS CLI commands to create and configure your S3 bucket:**

```bash
# 1. Create the S3 bucket (replace 'your-unique-terraform-state-bucket-name' and 'us-east-1')
aws s3 mb s3://your-unique-terraform-state-bucket-name --region us-east-1

# 2. Enable versioning on your bucket
aws s3api put-bucket-versioning \
    --bucket your-unique-terraform-state-bucket-name \
    --versioning-configuration Status=Enabled
```

Once your S3 bucket is ready, update the `bucket` attribute within the `backend "s3"` block in your **root `main.tf` file** to match your bucket's name:

```terraform
# main.tf (in the root directory of this repository)
terraform {
  backend "s3" {
    bucket         = "your-unique-terraform-state-bucket-name" # <--- ⚠️ UPDATE THIS LINE WITH YOUR S3 BUCKET NAME!
    key            = "ec2-instance-example/terraform.tfstate" # Path within the bucket
    region         = "us-east-1" # Ensure this matches your bucket's region
    encrypt        = true        # Ensure state file is encrypted at rest
  }
}
```

### 3\. Initialize Terraform

From the **root directory** of this repository, run `terraform init`. This command performs essential setup, including downloading the required AWS provider and configuring the S3 remote backend.

```bash
terraform init
```

### 4\. Plan & Apply Your Infrastructure

With initialization complete, you can now preview and apply your infrastructure changes.

  * **Review the execution plan:** Always run `terraform plan` first. This command shows you exactly what resources Terraform proposes to create, modify, or destroy based on your configuration and current state.

    ```bash
    terraform plan
    ```

    Carefully examine the output. You should see a plan indicating the creation of an `aws_instance` and an `aws_security_group`.

  * **Apply the configuration:** If the plan looks correct, proceed to apply the configuration. Terraform will then provision these resources in your AWS account.

    ```bash
    terraform apply --auto-approve
    ```

    *(**Note:** The `--auto-approve` flag automatically confirms the plan. For production or critical deployments, it's highly recommended to omit this flag and manually type `yes` after a thorough review of the plan.)*

Upon successful application, Terraform will display the output values (defined in `outputs.tf`), such as your new EC2 instance's ID and public IP address.

## 6\. Module Inputs

The root module (`main.tf`) consumes the `ec2-instance` module and provides the following variables for customization. These are defined in the `variables.tf` file within the root directory.

| Name                  | Description                                                                     | Type           | Default                     | Required |
| :-------------------- | :------------------------------------------------------------------------------ | :------------- | :-------------------------- | :------- |
| `instance_name`       | The name tag to apply to the EC2 instance.                                      | `string`       | `"my-test-instance"`        | no       |
| `ami_id`              | The AMI ID for the EC2 instance (e.g., a suitable Amazon Linux 2 AMI for `us-east-1`). | `string`       | `"ami-053b0d53c279acc90"`     | no       |
| `instance_type`       | The type of EC2 instance to provision. This variable is controlled to specific types. | `string`       | `"t2.micro"`                | no       |
| `ingress_ports`       | A list of inbound ports to open on the security group.                          | `list(number)` | `[22, 80, 443]`             | no       |
| `key_name`            | The name of an existing EC2 Key Pair to associate with the instance for SSH access. | `string`       | `null`                      | no       |
| `security_group_name` | The name for the created security group.                                        | `string`       | `null` (generates a name)   | no       |
| `region`              | The AWS region where all resources will be deployed.                            | `string`       | `"us-east-1"`               | no       |

## 7\. Module Outputs

Upon successful deployment, the root module's `outputs.tf` file provides the following easily accessible information about the provisioned resources:

| Name                    | Description                                       |
| :---------------------- | :------------------------------------------------ |
| `ec2_instance_id`       | The unique ID of the provisioned EC2 instance.    |
| `ec2_public_ip`         | The public IP address assigned to the EC2 instance.|
| `security_group_id`     | The ID of the created security group.             |
| `security_group_name`   | The name of the created security group.           |

You can retrieve these output values at any time from your terminal using:

```bash
terraform output
```

To get a specific output (e.g., just the public IP):

```bash
terraform output ec2_public_ip
```

## 8\. Cleaning Up Resources

To gracefully remove all resources provisioned by this Terraform configuration from your AWS account, execute the `terraform destroy` command:

```bash
terraform destroy --auto-approve
```

*(As always, consider removing `--auto-approve` for critical environments to ensure a confirmation prompt before deletion.)*

## 9\. ⚡ Important Note on State Locking ⚡

This configuration effectively utilizes an S3 bucket for remote Terraform state storage, offering significant benefits in terms of durability, versioning, and encryption for your `terraform.tfstate` file.

**However, it is crucial to understand that an S3 bucket, when used in isolation, DOES NOT inherently provide state locking.**

  * **The Risk:** In a collaborative or automated environment (e.g., CI/CD pipelines), if multiple users or processes attempt to run `terraform apply` concurrently against the *same state file*, it can lead to **severe state corruption**. This results in an inconsistent state file that misrepresents your actual infrastructure, potentially causing:
      * Errors during subsequent Terraform operations.
      * Creation of duplicate resources.
      * Unexpected deletion of existing resources.
      * "Phantom" resources (cloud resources not reflected in state, or vice-versa).
  * **The Recommendation:** For any team-based development, production deployments, or automated pipelines where state consistency is paramount, implementing a state locking mechanism is **highly recommended**. The most common and robust solution when using an S3 backend is to also provision and configure an **AWS DynamoDB table** specifically for state locking. DynamoDB provides a simple, consistent, and highly available lock, preventing simultaneous write operations to your state file.

While this example focuses on the core module and S3 for state storage, be acutely aware of this limitation in a multi-user context.

## 10\. Future Scope & Learning Journey

This project is a fundamental building block in our ongoing journey to master Terraform and IaC principles. Having moved past the basics of resource definition, we're now focusing on:

  * Developing more comprehensive and complex AWS infrastructure projects.
  * Targeting specific project requirements within various AWS US regions.
  * Integrating advanced best practices, including robust state locking with DynamoDB, to prepare for production-grade deployments.

Stay tuned for more updates as we continue to expand our IaC capabilities\!

## 11\. Contributing

Contributions, issues, and feature requests are always welcome\! If you have suggestions for improvements, find a bug, or want to expand upon this example, please feel free to:

1.  Fork the repository.
2.  Create your feature branch (`git checkout -b feature/your-awesome-feature`).
3.  Commit your changes (`git commit -m 'feat: Add an amazing new feature'`).
4.  Push to the branch (`git push origin feature/your-awesome-feature`).
5.  Open a Pull Request with a clear description of your changes.

## 12\. License

This project is open-sourced under the [MIT License](https://www.google.com/search?q=LICENSE). See the `LICENSE` file for more details.

## 13\. Contact

For any questions, feedback, or collaborations, feel free to reach out:

Nallagachu - [GitHub Profile](https://www.google.com/search?q=https://github.com/Nallagachu)

