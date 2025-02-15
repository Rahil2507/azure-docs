---
title: 'Customize a model with Azure OpenAI Service and Azure OpenAI Studio'
titleSuffix: Azure OpenAI
description: Learn how to create your own custom model with Azure OpenAI Service by using the Azure OpenAI Studio.
services: cognitive-services
manager: nitinme
ms.service: azure-ai-openai
ms.topic: include
ms.date: 09/01/2023
author: ChrisHMSFT
ms.author: chrhoder
keywords: 
---

## Prerequisites

- An Azure subscription. <a href="https://azure.microsoft.com/free/cognitive-services" target="_blank">Create one for free</a>.
- Access granted to Azure OpenAI in the desired Azure subscription.
- An Azure OpenAI resource that's located in a region that supports fine-tuning of the Azure OpenAI model. Check the [Model summary table and region availability](../concepts/models.md#model-summary-table-and-region-availability) for the list of available models by region and supported functionality. For more information, see [Create a resource and deploy a model with Azure OpenAI](../how-to/create-resource.md).

> [!NOTE]
> Currently, you must submit an application to access Azure OpenAI Service. To apply for access, complete [this form](https://aka.ms/oai/access). If you need assistance, open an issue on this repo to contact Microsoft.

## Review the workflow for Azure OpenAI Studio

Take a moment to review the fine-tuning workflow for using Azure OpenAI Studio with Azure OpenAI:

1. Prepare your training and validation data.
1. Use the **Create custom model** wizard in Azure OpenAI Studio to train your custom model.
    1. [Select a base model](#select-the-base-model).
    1. [Choose your training data](#choose-your-training-data).
    1. Optionally, [choose your validation data](#choose-your-validation-data).
    1. Optionally, [configure advanced options](#configure-advanced-options) for your fine-tune job.
    1. [Review your choices and train your new custom model](#review-your-choices-and-train-your-model).
1. Check the status of your custom model.
1. Deploy your custom model for use.
1. Use your custom model.
1. Optionally, analyze your custom model for performance and fit.

## Prepare your training and validation data

Your training data and validation data sets consist of input and output examples for how you would like the model to perform.

The training and validation data you use **must** be formatted as a JSON Lines (JSONL) document in which each line represents a single prompt-completion pair. The OpenAI command-line interface (CLI) includes [a data preparation tool](#openai-cli-data-preparation-tool) that validates, gives suggestions, and reformats your training data into a JSONL file ready for fine-tuning.

Here's an example of the training data format:

```json
{"prompt": "<prompt text>", "completion": "<ideal generated text>"}
{"prompt": "<prompt text>", "completion": "<ideal generated text>"}
{"prompt": "<prompt text>", "completion": "<ideal generated text>"}
```

In addition to the JSONL format, training and validation data files must be encoded in UTF-8 and include a byte-order mark (BOM). The file must be less than 200 MB in size. For more information about formatting your training data, see [Learn how to prepare your dataset for fine-tuning](../how-to/prepare-dataset.md).

### Create your training and validation datasets

Designing your prompts and completions for fine-tuning is different from designing your prompts for use with any of [our GPT-3 base models](../concepts/legacy-models.md#gpt-3-models). Prompts for completion calls often use either detailed instructions or few-shot learning techniques, and consist of multiple examples. For fine-tuning, each training example should consist of a single input prompt and its desired completion output. You don't need to give detailed instructions or multiple completion examples for the same prompt.

The more training examples you have, the better. It's a best practice to have at least 200 training examples. In general, doubling the dataset size leads to a linear increase in model quality.

For more information about preparing training data for various tasks, see [Learn how to prepare your dataset for fine-tuning](../how-to/prepare-dataset.md).

### OpenAI CLI data preparation tool

We recommend that you use OpenAI's CLI to assist with many of the data preparation steps. OpenAI has developed a tool that validates, gives suggestions, and reformats your data into a JSONL file ready for fine-tuning.

To install the OpenAI CLI, run the following Python command:

```console
pip install --upgrade openai 
```

To analyze your training data with the data preparation tool, run the following Python command. Replace the _\<LOCAL_FILE>_ argument with the full path and file name of the training data file to analyze:

```console
openai tools fine_tunes.prepare_data -f <LOCAL_FILE>
```

This tool accepts files in the following data formats, if they contain a prompt and a completion column/key:

- Comma-separated values (CSV)
- Tab-separated values (TSV)
- Microsoft Excel workbook (XLSX)
- JavaScript Object Notation (JSON)
- JSON Lines (JSONL)

After it guides you through the process of implementing suggested changes, the tool reformats your training data and saves output into a JSONL file ready for fine-tuning.

## Use the Create custom model wizard

Azure OpenAI Studio provides the **Create custom model** wizard, so you can interactively create and train a fine-tuned model for your Azure resource. 

1. Open Azure OpenAI Studio at <a href="https://oai.azure.com/" target="_blank">https://oai.azure.com/</a> and sign in with credentials that have access to your Azure OpenAI resource. During the sign-in workflow, select the appropriate directory, Azure subscription, and Azure OpenAI resource.

1. In Azure OpenAI Studio, browse to the **Management > Models** pane, and select **Create a custom model**.

   > [!NOTE]
   > If your resource doesn't have a model already deployed in it, OpenAI Studio displays a warning. For this exercise, you can ignore the warning because you fine-tune and deploy a new custom model.

   :::image type="content" source="../media/fine-tuning/studio-create-custom-model.png" alt-text="Screenshot that shows how to access the Create custom model wizard in Azure OpenAI Studio." lightbox="../media/fine-tuning/studio-create-custom-model.png":::

The **Create custom model** wizard opens.

### Select the base model

The first step in creating a custom model is to choose a base model. The **Base model** pane lets you choose a base model to use for your custom model. Your choice influences both the performance and the cost of your model.

Select the base model from the **Base model type** dropdown, and then select **Next** to continue.

You can create a custom model from one of the following available base models:
- `ada`
- `babbage`
- `curie`
- `code-cushman-001` (Currently unavailable for new customers)
- `davinci` (Currently unavailable for new customers)

For more information about our base models that can be fine-tuned, see [Models](../concepts/models.md).

:::image type="content" source="../media/fine-tuning/studio-base-model.png" alt-text="Screenshot that shows how to select the base model in the Create custom model wizard in Azure OpenAI Studio." lightbox="../media/fine-tuning/studio-base-model.png":::

### Choose your training data

The next step is to either choose existing prepared training data or upload new prepared training data to use when customizing your model. The **Training data** pane displays any existing, previously uploaded datasets and also provides options to upload new training data. 

:::image type="content" source="../media/fine-tuning/studio-training-data.png" alt-text="Screenshot of the Training data pane for the Create custom model wizard in Azure OpenAI Studio." lightbox="../media/fine-tuning/studio-training-data.png":::

- If your training data is already uploaded to the service, select **Choose dataset**.

   - Select the file from the list shown in the **Training data** pane.

- To upload new training data, use one of the following options:

   - Select **Local file** to [upload training data from a local file](#upload-training-data-from-local-file).

   - Select **Azure blob or other shared web locations** to [import training data from Azure Blob or another shared web location](#import-training-data-from-azure-blob-store).

For large data files, we recommend that you import from an Azure Blob store. Large files can become unstable when uploaded through multipart forms because the requests are atomic and can't be retried or resumed. For more information about Azure Blob Storage, see [What is Azure Blob Storage](../../../storage/blobs/storage-blobs-overview.md)?

> [!NOTE]
> Training data files must be formatted as JSONL files, encoded in UTF-8 with a byte-order mark (BOM). The file must be less than 200 MB in size.

#### Upload training data from local file

You can upload a new training dataset to the service from a local file by using one of the following methods:

- Drag and drop the file into the client area of the **Training data** pane, and then select **Upload file**.

- Select **Browse for a file** from the client area of the **Training data** pane, choose the file to upload from the **Open** dialog, and then select **Upload file**.

After you select and upload the training dataset, select **Next** to continue.

:::image type="content" source="../media/fine-tuning/studio-training-data-local.png" alt-text="Screenshot of the Training data pane for the Create custom model wizard, with local file options." lightbox="../media/fine-tuning/studio-training-data-local.png":::

#### Import training data from Azure Blob store

You can import a training dataset from Azure Blob or another shared web location by providing the name and location of the file.

1. Enter the **File name** for the file.

1. For the **File location**, provide the Azure Blob URL, the Azure Storage shared access signature (SAS), or other link to an accessible shared web location.

1. Select **Upload file** to import the training dataset to the service. 

After you select and upload the training dataset, select **Next** to continue.

:::image type="content" source="../media/fine-tuning/studio-training-data-blob.png" alt-text="Screenshot of the Training data pane for the Create custom model wizard, with Azure Blob and shared web location options." lightbox="../media/fine-tuning/studio-training-data-blob.png":::

### Choose your validation data

The next step provides options to configure the model to use validation data in the training process. If you don't want to use validation data, you can choose **Next** to continue to the advanced options for the model. Otherwise, if you have a validation dataset, you can either choose existing prepared validation data or upload new prepared validation data to use when customizing your model.

The **Validation data** pane displays any existing, previously uploaded training and validation datasets and provides options by which you can upload new validation data. 

:::image type="content" source="../media/fine-tuning/studio-validation-data.png" alt-text="Screenshot of the Validation data pane for the Create custom model wizard in Azure OpenAI Studio." lightbox="../media/fine-tuning/studio-validation-data.png":::

- If your validation data is already uploaded to the service, select **Choose dataset**.

   - Select the file from the list shown in the **Validation data** pane.

- To upload new validation data, use one of the following options:

   - Select **Local file** to [upload validation data from a local file](#upload-validation-data-from-local-file).
   
   - Select **Azure blob or other shared web locations** to [import validation data from Azure Blob or another shared web location](#import-validation-data-from-azure-blob-store).

For large data files, we recommend that you import from an Azure Blob store. Large files can become unstable when uploaded through multipart forms because the requests are atomic and can't be retried or resumed.

> [!NOTE]
> Similar to training data files, validation data files must be formatted as JSONL files, encoded in UTF-8 with a byte-order mark (BOM). The file must be less than 200 MB in size. 

#### Upload validation data from local file

You can upload a new validation dataset to the service from a local file by using one of the following methods:

- Drag and drop the file into the client area of the **Validation data** pane, and then select **Upload file**.

- Select **Browse for a file** from the client area of the **Validation data** pane, choose the file to upload from the **Open** dialog, and then select **Upload file**.

After you select and upload the validation dataset, select **Next** to continue.

:::image type="content" source="../media/fine-tuning/studio-validation-data-local.png" alt-text="Screenshot of the Validation data pane for the Create custom model wizard, with local file options." lightbox="../media/fine-tuning/studio-validation-data-local.png":::

#### Import validation data from Azure Blob store

You can import a validation dataset from Azure Blob or another shared web location by providing the name and location of the file.

1. Enter the **File name** for the file.

1. For the **File location**, provide the Azure Blob URL, the Azure Storage shared access signature (SAS), or other link to an accessible shared web location.

1. Select **Upload file** to import the training dataset to the service. 

After you select and upload the validation dataset, select **Next** to continue.

:::image type="content" source="../media/fine-tuning/studio-validation-data-blob.png" alt-text="Screenshot of the Validation data pane for the Create custom model wizard, with Azure Blob and shared web location options." lightbox="../media/fine-tuning/studio-validation-data-blob.png":::

### Configure advanced options

The **Create custom model** wizard shows hyperparameters for training your fine-tuned model on the **Advanced options** pane. The following hyperparameters are available:

:::image type="content" source="../media/fine-tuning/studio-advanced-options.png" alt-text="Screenshot of the Advanced options pane for the Create custom model wizard, with default options selected." lightbox="../media/fine-tuning/studio-advanced-options.png":::

Select **Default** to use the default values for the fine-tune job, or select **Advanced** to display and edit the hyperparameter values. 

The **Advanced** option lets you configure the following hyperparameters:

| Parameter name | Description |
| --- | --- |
| **Number of epochs** | The number of epochs to use for training the model. An epoch refers to one full cycle through the training dataset. |
| **Batch size** | The batch size to use for training. The batch size is the number of training examples used to train a single forward and backward pass. |
| **Learning rate multiplier** | The learning rate multiplier to use for training. The fine-tuning learning rate is the original learning rate used for pretraining, multiplied by this value. |
| **Prompt loss weight** | The weight to use for loss on the prompt tokens. This value controls how much the model tries to learn to generate the prompt (as compared to the completion, which always has a weight of 1.0). Increasing this value can add a stabilizing effect to training when completions are short. |

:::image type="content" source="../media/fine-tuning/studio-advanced-options-advanced.png" alt-text="Screenshot of the Advanced options pane for the Create custom model wizard, with advanced options selected." lightbox="../media/fine-tuning/studio-advanced-options-advanced.png":::

For more information about the hyperparameters, see [Create a Fine tune job](/rest/api/cognitiveservices/azureopenaistable/fine-tunes/create).

After you configure the advanced options, select **Next** to [review your choices and train your fine-tuned model](#review-your-choices-and-train-your-model).

### Review your choices and train your model

The **Review** pane of the wizard displays information about your configuration choices.

:::image type="content" source="../media/fine-tuning/studio-review.png" alt-text="Screenshot of the Review pane for the Create custom model wizard in Azure OpenAI Studio." lightbox="../media/fine-tuning/studio-review.png":::

If you're ready to train your model, select **Save and close** to start the fine-tune job and return to the **Models** pane. 

## Check the status of your custom model

The **Models** pane displays information about your custom model in the **Customized models** tab. The tab includes information about the status and job ID of the fine-tune job for your custom model. When the job completes, the tab displays the file ID of the result file.

:::image type="content" source="../media/fine-tuning/studio-models-job-running.png" alt-text="Screenshot of the Models pane from Azure OpenAI Studio, with a custom model displayed." lightbox="../media/fine-tuning/studio-models-job-running.png":::

After you start a fine-tune job, it can take some time to complete. Your job might be queued behind other jobs on the system. Training your model can take minutes or hours depending on the model and dataset size.

Here are some of the tasks you can do on the **Models** pane:

- Check the status of the fine-tune job for your custom model in the **Status** column of the **Customized models** tab.

- In the **Model name** column, select the model name to view more information about the custom model. You can see the status of the fine-tune job, training results, training events, and hyperparameters used in the job. 

- Select **Download training file** to download the training data you used for the model.

- Select **Download results** to download the result file attached to the fine-tune job for your model and [analyze your custom model](#analyze-your-custom-model) for training and validation performance.

- Select **Refresh** to update the information on the page. 

:::image type="content" source="../media/fine-tuning/studio-model-details.png" alt-text="Screenshot of the Models pane in Azure OpenAI Studio, with a custom model displayed." lightbox="../media/fine-tuning/studio-models-job-running.png":::

## Deploy a custom model

When the fine-tune job succeeds, you can deploy the custom model from the **Models** pane. You must deploy your custom model to make it available for use with completion calls.

[!INCLUDE [Fine-tuning deletion](fine-tune.md)]

> [!NOTE]
> Only one deployment is permitted for a custom model. An error message is displayed if you select an already-deployed custom model.

To deploy your custom model, select the custom model to deploy, and then select **Deploy model**.

:::image type="content" source="../media/fine-tuning/studio-models-deploy.png" alt-text="Screenshot that shows how to deploy a custom model in Azure OpenAI Studio." lightbox="../media/fine-tuning/studio-models-deploy.png":::

The **Deploy model** dialog box opens. In the dialog box, enter your **Deployment name** and then select **Create** to start the deployment of your custom model. 

:::image type="content" source="../media/fine-tuning/studio-models-deploy-model.png" alt-text="Screenshot of the Deploy Model dialog in Azure OpenAI Studio." lightbox="../media/fine-tuning/studio-models-deploy-model.png":::

You can monitor the progress of your deployment on the **Deployments** pane in Azure OpenAI Studio.

## Use a deployed custom model

After your custom model deploys, you can use it like any other deployed model. You can use the **Playground** pane in Azure OpenAI Studio to experiment with your new deployment. You can continue to use the same parameters with your custom model, such as `temperature` and `frequency penalty`, as you can with other deployed models. 

:::image type="content" source="../media/quickstarts/playground-load.png" alt-text="Screenshot of the Playground pane in Azure OpenAI Studio, with sections highlighted." lightbox="../media/quickstarts/playground-load.png":::

> [!NOTE]
> As with all applications, Microsoft requires a review process for your custom model before it's available live.

## Analyze your custom model

Azure OpenAI attaches a result file named _results.csv_ to each fine-tune job after it completes. You can use the result file to analyze the training and validation performance of your custom model. The file ID for the result file is listed for each custom model in the **Result file Id** column on the **Models** pane for Azure OpenAI Studio. You can use the file ID to identify and download the result file from the **File Management** pane of Azure OpenAI Studio. 

The result file is a CSV file that contains a header row and a row for each training step performed by the fine-tune job. The result file contains the following columns:

| Column name | Description |
| --- | --- |
| `step` | The number of the training step. A training step represents a single pass, forward and backward, on a batch of training data. |
| `elapsed_tokens` | The number of tokens the custom model has seen so far, including repeats. |
| `elapsed_examples` | The number of examples the model has seen so far, including repeats.<br>Each example represents one element in that step's batch of training data. For example, if the `Batch size` parameter is set to 32 in the [**Advanced options** pane](#configure-advanced-options), this value increments by 32 in each training step. |
| `training_loss` | The loss for the training batch. |
| `training_sequence_accuracy` | The percentage of completions in the training batch for which the model's predicted tokens exactly matched the true completion tokens.<br>For example, if the batch size is set to 3 and your data contains completions `[[1, 2], [0, 5], [4, 2]]`, this value is set to 0.67 (2 of 3) if the model predicted `[[1, 1], [0, 5], [4, 2]]`. |
| `training_token_accuracy` | The percentage of tokens in the training batch correctly predicted by the model.<br>For example, if the batch size is set to 3 and your data contains completions `[[1, 2], [0, 5], [4, 2]]`, this value is set to 0.83 (5 of 6) if the model predicted `[[1, 1], [0, 5], [4, 2]]`. |
| `validation_loss` | The loss for the validation batch. |
| `validation_sequence_accuracy` | The percentage of completions in the validation batch for which the model's predicted tokens exactly matched the true completion tokens.<br>For example, if the batch size is set to 3 and your data contains completions `[[1, 2], [0, 5], [4, 2]]`, this value is set to 0.67 (2 of 3) if the model predicted `[[1, 1], [0, 5], [4, 2]]`. |
| `validation_token_accuracy` | The percentage of tokens in the validation batch correctly predicted by the model.<br>For example, if the batch size is set to 3 and your data contains completions `[[1, 2], [0, 5], [4, 2]]`, this value is set to 0.83 (5 of 6) if the model predicted `[[1, 1], [0, 5], [4, 2]]`. |

## Clean up your deployments, custom models, and training files

When you're done with your custom model, you can delete the deployment and model. You can also delete the training and validation files you uploaded to the service, if needed. 

### Delete your model deployment

[!INCLUDE [Fine-tuning deletion](fine-tune.md)]

You can delete the deployment for your custom model on the **Deployments** pane in Azure OpenAI Studio. Select the deployment to delete, and then select **Delete** to delete the deployment. 

### Delete your custom model

You can delete a custom model on the **Models** pane in Azure OpenAI Studio. Select the custom model to delete from the **Customized models** tab, and then select **Delete** to delete the custom model.

> [!NOTE]
> You can't delete a custom model if it has an existing deployment. You must first [delete your model deployment](#delete-your-model-deployment) before you can delete your custom model.

### Delete your training files

You can optionally delete training and validation files that you uploaded for training, and result files generated during training, on the **Management** > **Data files** pane in Azure OpenAI Studio. Select the file to delete, and then select **Delete** to delete the file.

## Next steps

- Explore the fine-tuning capabilities in the [Azure OpenAI Service REST API reference](/azure/ai-services/openai/reference).
- Review [Python SDK operations](https://github.com/openai/openai-python/blob/main/examples/azure/finetuning.ipynb).
