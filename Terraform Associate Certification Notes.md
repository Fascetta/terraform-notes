# Terraform Associate Certification Notes

---

**About Me**

Hi!
I’m **Christian Bianchi**, a **Generative AI Engineer**, currently working at Storm Reply.
As someone passionate about learning and sharing knowledge, I’ve decided to start creating guides and notes on various topics I’m exploring in my professional journey. This is part of my commitment to improve my note-taking and share useful resources with others.
Feel free to reach out on [Linkedin](https://www.linkedin.com/in/christianbianchiit) if you have any questions, suggestions, or just want to connect!

---

# Introduction

Terraform is a powerful Infrastructure as Code (IaC) tool that enables developers and DevOps engineers to define, provision, and manage cloud infrastructure efficiently. Whether you're deploying on AWS, Azure, GCP, or other providers, Terraform simplifies infrastructure management with a declarative approach.

This guide provides a structured breakdown of key Terraform concepts, including variables, configuration blocks, essential commands, and best practices. By following these principles, you can improve automation, ensure infrastructure consistency, and maintain a scalable and maintainable cloud environment.

---

# I. Variables

## 1. Definining Variables

### Overview & Purpose

In Terraform, variables allow you to make configurations dynamic and reusable. Instead of hardcoding values, you define variables that can be assigned different values depending on the environment, user input, or external files. This enhances flexibility and maintainability across different deployments.

### Variable Types

Terraform supports different types of variables, categorized into primitive and complex types:

- Primitive Types
    - **string** – Represents a sequence of characters (e.g., "hello")
    - **number** – Represents numeric values (e.g., 42, 3.14)
    - **bool** – Represents boolean values (`true` or `false`)
    - **any** – A flexible type that can accept any value (primitive or complex)
- Complex Types
    - **list** – Ordered collection of values (e.g., `["a", "b", "c"]`)
    - **map** – Key-value pair collection (e.g., `{ "name": "John", "age": 30 }`)
    - **set** – Unordered collection of unique values (e.g., `set("apple", "banana")`)
    - **object** – Struct-like data structure defining multiple attributes with specific types
    - **tuple** – Ordered collection where each element has a predefined type

### Variable Validation Blocks

Terraform allows you to enforce constraints on variables using validation rules. This ensures that inputs meet expected conditions before being used in the configuration.

Example:

```hcl
variable "instance_count" {
  type = number
  validation {
    condition     = var.instance_count > 0 && var.instance_count <= 10
    error_message = "Instance count must be between 1 and 10."
  }
}
```

---

## 2. Sensitive Information in Variables

### Handling Secrets Securely

Terraform variables can contain sensitive data such as API keys, passwords, and tokens. To avoid exposing these values in logs or outputs, Terraform provides the `sensitive` attribute.

Example:

```hcl
variable "db_password" {
  type      = string
  sensitive = true
}
```

This prevents the value from being displayed in Terraform logs or outputs.

### Managing Sensitive Data

There are multiple ways to manage sensitive data securely in Terraform:

1. **Using `.tfvars` files** – Define sensitive values in separate files and reference them in your configuration:
    
    ```bash
    db_password = "supersecurepassword"
    ```
    
    Then load the file using:
    
    ```bash
    terraform apply -var-file="secrets.tfvars"
    ```
    
2. **Using Environment Variables** – Terraform automatically picks up environment variables prefixed with `TF_VAR_`:
    
    ```bash
    export TF_VAR_db_password="supersecurepassword"
    ```
    

---

## 3. Output Variables & Local Values

### Output Variables Overview

Output variables in Terraform allow you to extract and display important values from your infrastructure. They can also be used for passing values to external tools.

Example:

```hcl
output "instance_ip" {
  value = aws_instance.my_instance.public_ip
}
```

This will display the instance’s public IP after Terraform execution.

### Defining and Using Local Values

Local values are used to define reusable expressions that simplify Terraform configurations. Unlike variables, they cannot be overridden but help in improving code readability.

Example:

```hcl
locals {
  common_tags = {
    Project   = "MyApp"
    Owner     = "DevOps Team"
  }
}

resource "aws_instance" "example" {
  tags = local.common_tags
}
```

This avoids repetition and ensures consistent tagging across resources.

---

# II. Blocks

## 1. Terraform Provider Blocks

### The `terraform` Block & `required_providers`

The `terraform` block is used to define Terraform settings and configurations, including required providers. The `required_providers` block specifies which providers your configuration depends on and allows you to define provider versions.

**Example:**

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

This ensures Terraform pulls the correct provider version from the Terraform Registry.

### Version Constraints and Aliases

Version constraints specify which versions of a provider are acceptable. They prevent breaking changes by restricting automatic upgrades.

- `~>` allows updates within a minor version.
- `>=` specifies a minimum version.
- `<=` specifies a maximum version.

Aliases enable multiple configurations of the same provider in different regions.

**Example:**

```hcl
provider "aws" {
  region = "us-east-1"
  alias  = "east"
}
```

---

## 2. Resource Blocks

### Defining Resources

A **resource block** defines an infrastructure object in Terraform, such as a virtual machine, database, or network component.

**Example:**

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
}
```

Here, an AWS EC2 instance is created using the specified AMI and instance type.

### Attributes and Dependencies

Terraform automatically determines dependencies, but explicit dependencies can be set using `depends_on`.

**Example:**

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
  depends_on    = [aws_security_group.web_sg]
}
```

This ensures that the security group is created before the instance.

### Lifecycle Rules

Terraform provides lifecycle rules to control resource behavior during updates.

- `create_before_destroy`: Ensures a new resource is created before the old one is destroyed.
- `prevent_destroy`: Prevents accidental deletion.
- `ignore_changes`: Ignores specific attribute changes.

**Example:**

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"

  lifecycle {
    prevent_destroy = true
    ignore_changes  = [ami]
  }
}
```

### Resource Targeting

Terraform allows targeting specific resources during an operation.

**Example:**

```bash
tf apply -target=aws_instance.web
```

This applies changes only to `aws_instance.web`.

---

## 3. Data Source Blocks

### Overview of Data Sources

Data sources allow Terraform to retrieve external information without creating new infrastructure.

**Example:**

```hcl
data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*"]
  }
}
```

This retrieves the latest Amazon Linux AMI.

---

## 4. Module Blocks

### Root vs. Child Modules

Modules help organize Terraform configurations. The **root module** is the main entry point, while **child modules** are reusable components.

**Example Usage of a Module:**

```hcl
module "network" {
  source = "./modules/network"
  vpc_id = "vpc-abc123"
}
```

This calls a child module located in `./modules/network`.

### Sourcing Modules from the Terraform Registry

Modules can be sourced from the Terraform Registry, Git repositories, or local directories.

**Example:**

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.0.0"
  cidr    = "10.0.0.0/16"
}
```

This pulls a prebuilt VPC module from the Terraform Registry.

---

## 5. Dynamic Blocks & Splat Expressions

### Creating Nested Blocks Programmatically

Dynamic blocks allow for flexible resource definitions.

**Example:**

```hcl
resource "aws_security_group" "example" {
  dynamic "ingress" {
    for_each = [
      { from_port = 80, to_port = 80, protocol = "tcp" },
      { from_port = 443, to_port = 443, protocol = "tcp" }
    ]
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

This dynamically generates ingress rules for both HTTP and HTTPS.

### Using Splat Syntax for Lists and Maps

Splat expressions simplify list and map access.

**Example:**

```hcl
output "instance_ids" {
  value = aws_instance.web[*].id
}
```

This retrieves all instance IDs in a list format.

---

# III. Commands

## 1. Core Terraform Workflow Commands

### `terraform init`

Initializes a Terraform working directory by downloading required provider plugins and setting up the backend configuration. This command should always be run first before executing any other Terraform commands.

### `terraform plan`

Generates an execution plan showing what actions Terraform will take to achieve the desired state. It helps preview changes without applying them.

### `terraform apply`

Applies the planned changes to the infrastructure. This command executes resource creation, modification, and deletion based on the configuration files.

Can be run without confimation using the `-auto-approve` flag.

### `terraform destroy`

Destroys all resources managed by the Terraform configuration. It removes infrastructure components to avoid unnecessary costs.

---

## 2. Utility Commands

### `terraform validate`

Checks the syntax and validity of Terraform configuration files to ensure they are correctly formatted and error-free.

### `terraform fmt`

Formats Terraform configuration files to match the standard style, improving readability and consistency.

### `terraform graph`

Generates a dependency graph of the Terraform resources and outputs it in DOT format, which can be visualized using graphing tools (like Graphviz).

### `terraform providers`

Displays the providers used in the current configuration, including their versions and sources.

### `terraform show`

Outputs human-readable information about the current Terraform state or a specific plan file.

Using option `-json` will display the details in json format.

### `terraform output`

Retrieves and displays the output values from the Terraform state, which can be used to extract important data such as generated IP addresses or resource IDs.

Running `terraform output` print all outputs, while `terraform output <name>` prints a specific one.

---

## 3. State Management Commands

### `terraform state list`

Lists all resources currently tracked in the Terraform state.

### `terraform state mv`

Moves a resource within the state file to a new name or location, useful when refactoring configurations.

### `terraform state pull`

Retrieves the latest Terraform state file from remote storage.

### `terraform state rm`

Removes a specific resource from the Terraform state without deleting the actual infrastructure.

### `terraform state show`

Displays detailed information about a specific resource in the Terraform state.

## 4. Workspaces & Resource Management

### Workspace Commands

- `terraform workspace list` – Lists all available workspaces.
- `terraform workspace new <name>` – Creates a new workspace.
- `terraform workspace select <name>` – Switches to a specified workspace.

### Count and `for_each` Usage

- `count` – Used to create multiple instances of a resource dynamically.
- `for_each` – Allows iteration over maps and sets for more complex resource provisioning.

### Taint and Untaint Commands

- `terraform taint <resource>` – Marks a resource for recreation during the next `apply`.
- `terraform untaint <resource>` – Removes the taint, preventing unnecessary recreation.

---

## 5. Provisioners & Debugging

### Provisioners

Provisioners are used to execute scripts or commands on a resource after it is created.

- **Remote Exec** – Executes commands on a remote machine via SSH or WinRM.
- **Local Exec** – Runs a command on the machine executing Terraform.
- **Destroy-time Provisioners** – Executes commands before a resource is destroyed.

### Debugging Tools

- **`TF_LOG`** – Enables logging at different verbosity levels (DEBUG, INFO, WARN, ERROR) to troubleshoot issues.
- **`TF_LOG_PATH`** – Specifies a file to store logs for further analysis.

---

# IV. Terraform State & Remote State

## 1. Terraform State Overview

Terraform maintains a state file that acts as a source of truth for the infrastructure it manages. This file maps Terraform resources to real-world infrastructure components, helping Terraform track changes and determine what actions are needed during execution.

### Structure of the State File

The Terraform state file (`terraform.tfstate`) is a JSON document that contains:

- A mapping of resources to their real-world counterparts.
- Metadata about resource dependencies.
- Computed values for attributes (e.g., IP addresses, instance IDs).
- Terraform version and provider information.

### Best Practices for State Storage

- **Use Remote Storage:** Avoid storing the state file locally to prevent accidental loss or corruption.
- **Enable State Locking:** Prevent concurrent Terraform executions that could corrupt the state file.
- **Encrypt the State File:** When stored remotely, ensure encryption (e.g., S3 bucket with SSE, GCS with KMS, or Azure Blob encryption).
- **Use Version Control Sparingly:** Do not commit the state file to Git; instead, use remote storage with versioning enabled.
- **Regularly Backup State:** Implement automated state backups to recover from corruption or accidental deletions.

### Considerations for Automatic State Refresh

Terraform automatically refreshes state when executing `terraform plan` or `terraform apply`. However, automatic state refresh can sometimes cause:

- **Performance Issues:** Refreshing state can be slow if Terraform is managing a large number of resources.
- **Unexpected Changes:** If infrastructure changes outside of Terraform, a refresh may show drift that requires manual intervention.
- **Inconsistent Environments:** Running Terraform in different environments without proper state management can lead to resource conflicts.

To manually refresh state, use:

```bash
terraform refresh
```

---

## 2. Remote State

Terraform allows storing state remotely instead of locally to support collaboration and ensure consistency across teams.

### Benefits of Remote State

- **Collaboration:** Multiple team members can work with the same state without conflicts.
- **State Locking:** Prevents simultaneous Terraform operations that could corrupt state.
- **Security:** Remote backends provide access control and encryption.
- **Versioning & Backup:** Many remote backends support automatic versioning, allowing rollback if needed.

### Setting Up a Remote Backend

Terraform supports multiple backends for remote state storage, including:

### AWS S3

To store state in an S3 bucket with state locking using DynamoDB:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "global/s3/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}
```

### Azure Blob Storage

To store state in an Azure Blob Storage account:

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "my-resource-group"
    storage_account_name = "mystorageaccount"
    container_name       = "terraform-state"
    key                  = "terraform.tfstate"
  }
}
```

### Google Cloud Storage (GCS)

To store state in a GCS bucket:

```hcl
terraform {
  backend "gcs" {
    bucket = "my-terraform-state"
    prefix = "terraform/state"
  }
}
```

### State Locking

State locking prevents multiple users from modifying the same state file simultaneously. AWS DynamoDB is commonly used for locking when using S3 as the backend.

To set up DynamoDB for state locking:

1. Create a DynamoDB table:
    
    ```bash
    aws dynamodb create-table \
      --table-name terraform-lock \
      --attribute-definitions AttributeName=LockID,AttributeType=S \
      --key-schema AttributeName=LockID,KeyType=HASH \
      --billing-mode PAY_PER_REQUEST
    ```
    
2. Configure Terraform to use this table for state locking (as shown in the S3 backend example above).

Other remote backends such as HashiCorp Terraform Cloud also provide built-in state locking and versioning.

By following these practices, Terraform users can manage state effectively, ensuring security, reliability, and consistency across their infrastructure deployments.

---

# V. Functions & Expressions

Terraform provides a set of built-in functions that help manipulate and transform data within configurations. These functions operate on different data types, such as strings, numbers, lists, and maps, making infrastructure code more dynamic and reusable.

## 1. Terraform Built-in Functions

### **File and Length Functions**

- **`file(path)`**: Reads the contents of a file at the specified path.
    - Example:
        
        ```hcl
        content = file("config.json")
        ```
        
- **`length(value)`**: Returns the number of elements in a list, map, or string.
    - Example:
        
        ```hcl
        count = length(["aws", "azure", "gcp"])  # Output: 3
        ```
        
- **`fileexists(path)`**: Returns `true` if the specified file exists.
    - Example:
        
        ```hcl
        exists = fileexists("secrets.tfvars")
        ```
        

### **String Functions**

Terraform provides several string manipulation functions to modify text values dynamically.

- **`lower(string)`**: Converts a string to lowercase.
    - Example:
        
        ```hcl
        name = lower("Hello World")  # Output: "hello world"
        ```
        
- **`upper(string)`**: Converts a string to uppercase.
    - Example:
        
        ```hcl
        name = upper("terraform")  # Output: "TERRAFORM"
        ```
        
- **`title(string)`**: Capitalizes the first letter of each word.
    - Example:
        
        ```hcl
        formatted_name = title("hello world")  # Output: "Hello World"
        ```
        
- **`split(delimiter, string)`**: Splits a string into a list based on a delimiter.
    - Example:
        
        ```hcl
        list = split(",", "aws,azure,gcp")  # Output: ["aws", "azure", "gcp"]
        ```
        
- **`join(delimiter, list)`**: Joins elements of a list into a single string with a delimiter.
    - Example:
        
        ```hcl
        str = join("-", ["aws", "azure", "gcp"])  # Output: "aws-azure-gcp"
        ```
        
- **`substr(string, offset, length)`**: Extracts a substring from a given string.
    - Example:
        
        ```hcl
        short = substr("terraform", 0, 4)  # Output: "terr"
        ```
        

### **Collection Functions**

Terraform also includes functions to work with lists and maps, helping with data structure manipulations.

- **`contains(list, value)`**: Returns `true` if the value exists in the list.
    - Example:
        
        ```hcl
        exists = contains(["aws", "azure", "gcp"], "aws")  # Output: true
        ```
        
- **`index(list, value)`**: Returns the index of a value in a list.
    - Example:
        
        ```hcl
        idx = index(["aws", "azure", "gcp"], "azure")  # Output: 1
        ```
        
- **`keys(map)`**: Returns a list of all keys in a map.
    - Example:
        
        ```hcl
        envs = keys({ dev = "AWS", prod = "Azure" })  # Output: ["dev", "prod"]
        ```
        
- **`values(map)`**: Returns a list of all values in a map.
    - Example:
        
        ```hcl
        clouds = values({ dev = "AWS", prod = "Azure" })  # Output: ["AWS", "Azure"]
        ```
        
- **`lookup(map, key, default)`**: Retrieves the value of a key in a map, returning a default value if the key is not found.
    - Example:
        
        ```hcl
        region = lookup({ dev = "us-east-1", prod = "us-west-2" }, "dev", "us-central-1")  # Output: "us-east-1"
        ```
        

### **Operators and Conditional Expressions**

Terraform supports logical operations and conditionals for decision-making.

- **Comparison Operators**: Terraform supports standard comparison operators:
    - `==` (equal), `!=` (not equal), `>` (greater than), `<` (less than), `>=`, `<=`
- **Logical Operators**:
    - `&&` (AND), `||` (OR), `!` (NOT)
- **Conditional Expressions (`? :` syntax)**
    
    Terraform supports ternary expressions, useful for conditional assignments.
    
    - Example:
    
    If `var.is_prod` is `true`, `env` will be `"production"`, otherwise `"development"`.
        
        ```hcl
        env = var.is_prod ? "production" : "development"
        ```
        
- **Coalescing (`coalesce` function)**
    
    Returns the first non-null value in a list of arguments.
    
    - Example:
        
        ```hcl
        result = coalesce(null, "", "default_value")  # Output: "default_value"
        ```
        

---

# VI. Dependency Lock File

### 1. Purpose and Importance of `terraform.lock.hcl`

The `terraform.lock.hcl` file is a dependency lock file introduced in Terraform to ensure consistent provider versions across different environments and team members. It helps prevent unexpected updates that could introduce breaking changes.

Key benefits of using the lock file:

- **Ensures reproducibility**: Keeps the same provider versions even if new versions are available.
- **Improves stability**: Reduces the risk of breaking infrastructure due to provider changes.
- **Enhances collaboration**: Team members working on the same project will use identical provider versions.
- **Secures the infrastructure**: Prevents unintentional upgrades that might introduce vulnerabilities or compatibility issues.

By default, Terraform automatically generates the `terraform.lock.hcl` file when you run `terraform init`.

---

### 2. Upgrading the Lock File

While Terraform locks provider versions, you may need to upgrade them intentionally when a newer version is required. To upgrade providers:

1. **Manually edit the provider version constraint** in the `required_providers` block of your configuration file (`.tf` files).
2. Run the following command to upgrade providers:
    
    ```bash
    terraform init -upgrade
    ```
    
    This will:
    
    - Fetch the latest compatible provider versions (based on the constraints in `required_providers`).
    - Update the `terraform.lock.hcl` file with the new checksums and versions.
3. To verify the update, use:
    
    ```bash
    terraform providers
    ```
    
    This command lists the active provider versions in use.
    

---

### 3. Best Practices for Managing Provider Versions

1. **Always use version constraints in `required_providers`**
    
    Example:
    
    ```hcl
    terraform {
      required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "~> 5.0"  # Allows upgrades within the 5.x range but not 6.x
        }
      }
    }
    ```
    
    This prevents Terraform from automatically pulling breaking changes from major releases.
    
2. **Commit the `terraform.lock.hcl` file to version control**
    - This ensures all team members use the same provider versions.
    - Avoid adding `terraform.lock.hcl` to `.gitignore` unless there’s a specific reason.
3. **Periodically review and upgrade providers**
    - Running `terraform init -upgrade` at regular intervals ensures you stay up to date with the latest compatible provider improvements and security patches.
4. **Use a remote backend for Terraform state**
    - If your team collaborates on infrastructure, store the state file remotely to prevent conflicts.

By following these practices, you maintain stability while keeping your Terraform environment up to date with necessary improvements.

---

# Conclusion

I hope this guide has helped you get a solid understanding of Terraform and its key concepts. Whether you’re looking to manage infrastructure more efficiently or just getting started with cloud technologies, Terraform is a powerful tool that can help streamline your workflows.

As I continue to explore new topics, I’ll be sharing more guides and notes to help others along the way. If you have any feedback, questions, or suggestions, don’t hesitate to reach out. Let’s learn and grow together!

Feel free to connect with me on [LinkedIn](https://www.linkedin.com/in/christianbianchiit), where I share my ongoing learning journey and connect with other like-minded professionals.