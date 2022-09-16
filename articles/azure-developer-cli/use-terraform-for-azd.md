---
title: Use Terraform as an infrastructure as code tool for Azure Developer CLI (azd) Preview 
description: How to use Terraform as an infrastructure as code tool for Azure Developer CLI (azd) Preview.
author: hhunter-ms
ms.author: hannahhunter
ms.date: 09/14/2022
ms.service: azure-dev-cli
ms.topic: conceptual
ms.custom: devx-track-azdevcli
---

# Use Terraform as an infrastructure as code tool for Azure Developer CLI (azd) Preview

Azure Developer CLI (azd) Preview supports multiple infrastructure as code (IaC) providers, including:  
- [Bicep](/azure/azure-resource-manager/bicep/overview?tabs=bicep)
- [Terraform](/azure/developer/terraform/overview)

By default, `azd` assumes Bicep as the IaC provider. Refer to the [Comparing Terraform and Bicep](/azure/developer/terraform/comparing-terraform-and-bicep?tabs=comparing-bicep-terraform-integration-features) article for help deciding which IaC provider is best for your project.

## Configure Terraform as the IaC provider

1. If you haven't already, [install and configure Terraform](/azure/developer/terraform/quickstart-configure?source=recommendations).
1. Open the `azure.yaml` file found in the root of your project and make sure you have the following lines to override the default, which is Bicep:

    ```yaml
    infra:
      provider: terraform
    ```

1. Add all your `.tf` files to the `infra` directory found in the root of your project.
1. Run `azd up`. 
   
> [!NOTE] 
> Check out these two azd templates with Terraform as IaC Provider: [Node.js and Terraform](https://github.com/Azure-Samples/todo-nodejs-mongo-terraform) and [Python and Terraform](https://github.com/Azure-Samples/todo-python-mongo-terraform). 

## azd pipeline config for Terraform

Terraform stores state about your managed infrastructure and configuration. Because of this state file, you need to enable remote state **before** you run `azd pipeline config` to set up your deployment pipeline in GitHub.

By default, `azd` assumes the use of local state file. If you ran `azd up` before enabling remote state, you need to run `azd down` and switch to remote state file.

## Local vs remote state

Terraform uses persisted [state](https://www.terraform.io/language/state) data to keep track of the resources it manages. 

Scenarios for enabling remote state:

- To allow shared access to the state data, and allow multiple people work together on that collection of infrastructure resources
- To avoid exposing sensitive information included in state file
- To decrease the chance of inadvertent deletion because of storing state locally

## Enable remote state

1. Make sure you [configure a remote state storage account](/azure/developer/terraform/store-state-in-azure-storage).
1. Add a new file called `provider.conf.json` in the `infra` folder.

    ```json
    {
        "storage_account_name": "${RS_STORAGE_ACCOUNT}",
        "container_name": "${RS_CONTAINER_NAME}",
        "key": "azd/azdremotetest.tfstate",
        "resource_group_name": "${RS_RESOURCE_GROUP}"
    }
    ```

1. Update `provider.tf` found in the `infra` folder to set the backend to be remote

    ```console
    # Configure the Azure Provider
    terraform {
      required_version = ">= 1.1.7, < 2.0.0"
      backend "azurerm" {
      }
    ```

1. Run `azd env set <key> <value>` to add configuration in the `.env` file. 
For example: 
 
    ```bash
    azd env set RS_STORAGE_ACCOUNT your_storage_account_name
    azd env set RS_CONTAINER_NAME your_terraform_container_name
    azd env set RS_RESOURCE_GROUP your_storage_account_resource_group
    ```

1. Run the next `azd` command as per your usual workflow. When remote state is detected, `azd` initializes Terraform with the configured backend configuration.

1. To share the environment with teammates, make sure they run `azd env refresh -e <environmentName>` to refresh environment settings in the local system, and perform Step 4 to add configuration in the `.env` file.

## See also

- For more on remote state, see [store Terraform state in Azure Storage](/azure/developer/terraform/store-state-in-azure-storage).
- Template: [Todo Application with Node.js and Terraform](https://github.com/Azure-Samples/todo-nodejs-mongo-terraform)
- Template: [Todo Application with Python and Terraform](https://github.com/Azure-Samples/todo-python-mongo-terraform)

## Next steps

> [!div class="nextstepaction"]
> [Azure Developer CLI FAQ](./faq.yml)