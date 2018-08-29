# Get Azure Resource Manager context at runtime

A key benefit of Azure is consistency. Development investments for one location are reusable in another. A template makes your deployments consistent and repeatable across environments, including the global Azure, Azure sovereign clouds, and Azure Stack. However, even though the global, sovereign, hosted, and hybrid clouds provide consistent services, not all clouds are identical. 

Since update [1807](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-update-1807) of Azure Stack, all cloud support a condition property for resources. This condition can be use to deploy a resource based an the evaluation of a [logical function](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-functions-logical).

The sample in this repository provides the context of the environment the template is being deployed to at runtime. By deploying a storage account, retrieving the endpoint from that storage account and evaluating it against a set of values, it is determined if the deployment is executed on Global Azure, Azure German Cloud, Azure China Cloud, Azure Government Cloud, or Azure Stack. Based on the environment you can set a condition on a resource that can be deployed or skipped. For example, if you have an optional resource that relies on a resource provider that does not exist in Azure Stack, you can set a condition on that resource and only deploy it if the context is not AzureStack.

## How to use this example
You can reference the linked template from your own deployment template by adding a linked deployment resource to your template

``` json
    "resources": [
        {
            "name": "getContext",
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2015-01-01",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                    "uri": "https://raw.githubusercontent.com/marcvaneijk/azure-resource-manager-context/master/linked/context.json",
                    "contentVersion": "1.0.0.0"
                }
            }
        } 
```

The output from the linked template return on of the following values (baes on the environment the deployment is executed against)

- AzureCloud
- AzureChinaCloud
- AzureGermanCloud
- AzureUSGovernment
- AzureStack

You can reference the output from the linked template by referencing the output in your main template.

``` json
    "outputs": {
        "cloud": {
            "value": "[reference('getContext').outputs.cloud.value]",
            "type": "string"
        }
    }
```
For example, if you want to skip a resource deployment on Azure Stack, but deploy the resource on other clouds your resource configuration requires the following condition
``` json
    "resources": [
        {
            "condition": "[not(equals(reference('getContext').outputs.cloud.value,'AzureStack'))]",
            "type": "Microsoft.Automation",
            "name": "[variables('runBookName')]",
```
The templates in this reposiotry are only provided as an example.
The storage account created in the linked template is only used for retrieving the endpoint. You can copy/fork the templates and make changes to it to better fit your need.


