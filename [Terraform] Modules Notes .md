# What are modules for?

Here are some of the ways that modules help solve the problems listed above:

- **Organize Configuration**:  
  - Group related parts of your configuration together for easier navigation, understanding, and updates.  
  - Simplify management of even moderately complex infrastructure with hundreds or thousands of lines of configuration.  

- **Encapsulate Configuration**:  
  - Encapsulate configuration into distinct logical components to reduce unintended consequences.  
  - Prevent accidental changes to other infrastructure and minimize errors like duplicate resource names.  

- **Reuse Configuration**:  
  - Save time and reduce errors by reusing configuration written by yourself, your team, or the Terraform community.  
  - Share your modules to enable others to benefit from your work.  

- **Provide Consistency and Ensure Best Practices**:  
  - Promote consistency across configurations, making them easier to understand.  
  - Ensure best practices are applied by creating modules for specific use cases, such as public website buckets or private logging buckets.  
  - Update configurations centrally when needed.  

- **Benefits of Using Modules**:  
  - Simplify complex configurations.  
  - Reduce errors.  
  - Ensure a consistent approach to managing infrastructure.  

Using modules can help reduce these errors. For example, you might create a module to describe how all of your organization's public website buckets will be configured, and another module for private buckets used for logging applications. Also, if a configuration for a type of resource needs to be updated, using modules allows you to make that update in a single place and have it be applied to all cases where you use that module.

## Module Structure

Terraform treats any local directory referenced in the source argument of a module block as a module. A typical file structure for a new module is:

```plaintext
├── LICENSE
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf
```

> **Note:** None of these files are required or have any special meaning to Terraform when it uses your module. You can create a module with a single `.tf` file or use any other file structure you like.

Each of these files serves a purpose:

- **`LICENSE`**: Contains the license under which your module will be distributed. When you share your module, the `LICENSE` file informs users of the terms under which it has been made available. Terraform itself does not use this file.
- **`README.md`**: Contains documentation in markdown format that describes how to use your module. Terraform does not use this file, but services like the Terraform Registry and GitHub will display its contents to visitors.
- **`main.tf`**: Contains the main set of configurations for your module. You can also create other configuration files and organize them in a way that makes sense for your project.
- **`variables.tf`**: Contains the variable definitions for your module. When your module is used by others, the variables will be configured as arguments in the module block. Variables without default values become required arguments, while those with default values can be overridden.
- **`outputs.tf`**: Contains the output definitions for your module. Module outputs are made available to the configuration using the module, often used to pass information about the infrastructure defined by the module to other parts of your configuration.

### Files to Exclude from Distribution

Be aware of these files and ensure that you don't distribute them as part of your module:

- **`terraform.tfstate`** and **`terraform.tfstate.backup`**: These files contain your Terraform state and track the relationship between your configuration and the provisioned infrastructure.
- **`.terraform` directory**: Contains the modules and plugins used to provision your infrastructure. These files are specific to an individual Terraform instance.
- **`*.tfvars` files**: These files don't need to be distributed unless you are also using the module as a standalone Terraform configuration. Module input variables are typically set via arguments in the module block.

> **Note:** If you are tracking changes to your module in a version control system such as Git, configure your version control system to ignore these files. For an example, see this [`.gitignore` file from GitHub](https://github.com/github/gitignore/blob/main/Terraform.gitignore).

### Install the local module

Whenever you add a new module to a configuration, Terraform must install the module before it can be used. Both the terraform get and terraform init commands will install and update modules. The terraform init command will also initialize backends and install plugins.

1. Install the module:

    ```bash
    terraform init
    ```

2. Provision your bucket:

    ```bash
    terraform apply
    ```

3. Respond yes to the prompt. Your bucket and other resources will be provisioned.

### Simple command to create file and its content

Run the following command to create a file called README.md with the following content:

```bash
tee -a README.md <<EOF
# GCS static website bucket

This module provisions Cloud Storage buckets configured for static website hosting.
EOF
```
