# Querying and Analyzing Logs from Project Bonsai

## Description

Welcome! The notebooks in this repository aim to provide templates for importing and visualizing logs for brains trained or assessed using the [Bonsai platform](https://docs.microsoft.com/en-us/bonsai/). The data is imported from an Azure Log Analytics Workspace (LAW) and into a Jupyter notebook.

Training and assessment logs are saved in an automatically provisioned [Azure Log Analytics workspace](https://docs.microsoft.com/en-us/azure/azure-monitor/log-query/log-analytics-tutorial) (and can be queried via [KQL](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/tutorial?pivots=azuremonitor) language). By default, four simulators are logged automatically during assessment. To collect logs from more simulators during training, use the Bonsai CLI and refer to the instructions below.

This [documentation](https://docs.microsoft.com/en-us/bonsai/cli/brain/version/start-logging) offers more details on enabling logging via the CLI.

## Prerequisites

1. Install the package requirements: `pip install -r requirements.txt`
2. Train a brain and start logging using instructions below
 
You can generate logs from brains that utilize managed or unmanaged simulators. We outline both approaches below:

## Logging training (with unmanaged sims)

1. Start an unmanaged sim and brain training as you normally would: 
    1. Register a sim by launching your sim. For example `python main.py` (or through our partner sims AnyLogic or Simulink)
    1. Start brain training `bonsai brain version start-training --name <BRAIN_NAME>`
    1. Connect your registered sim to a brain `bonsai simulator connect --simulator-name <SIM_NAME> --brain-name <BRAIN_NAME> --version <VERSION_#> --action Train --concept-name <CONCEPT_NAME>`
2. Find the session-id of un-managed sim using Bonsai CLI: `bonsai simulator unmanaged list`
3. When you're ready to [start logging](https://docs.microsoft.com/en-us/bonsai/cli/brain/version/start-logging):
`bonsai brain version start-logging -n <BRAIN_NAME> --session-id <SESSION_ID>`
    1.Note: A Log Analytics workspace will be provisioned automatically on Azure if it does not already exist
4. You can find the Log Analytics workspace id on portal.azure.com. It will be created under your provisioned resource group `bonsai-rg-<BONSAI WORKSPACE NAME >-<WORKSPACE-ID>`
    1. Note: It might take ~5 minutes to show up if it is the first time using logging feature as the log analytics workspace gets created.
5. Logs will start populating the Log Analytics workspace 3-5 minutes after starting logging
    1. Note: if this is the first time you're using logging you may need to wait for the first episode to finish so that episode-level (EpisodeLog_CL table) logs gets created and filled with at least 1 row of data.
    1. Optional: Navigate to https://ms.portal.azure.com/#blade/Microsoft_Azure_Monitoring_Logs/LogsBlade to query logs in the webUI. Sample KQL query, take 10 samples of IterationLog_CL table for the corresponding sim's session-id:
    ```KQL
    IterationLog_CL
    | where SessionId_s == <SESSION_ID>
    | take 10
    ```
6. Open the jupyter notebook `01-retrieving-logs.ipynb` to perform sample queries, or query using the log analytics workspace environment.
    - start jupyter: ```jupyter notebook```

> **WARNING**: Be careful, or else!


## Logging training (with managed sims)

1. start brain training `bonsai brain version start-training --name <BRAIN_NAME>`
2. When you're ready to [start logging](https://docs.microsoft.com/en-us/bonsai/cli/brain/version/start-logging):
`bonsai brain version start-logging -n <brain-name> --version <version-number> -m`
    1. Note: A Log Analytics workspace will be provisioned automatically on Azure if it does not already exist
3. Refer to the noteboook `02-custom-assessment-logs.ipynb` to view some sample queries, or perform queries using the log analytics workspace environment.

## Logging custom assessements

1. Start a custom assessment (with a managed or unmanaged sim) as you usually would, using either the CLI or the web UI
2. Wait for your assessment to start running (in the UI, you can tell that your assessment is running when you click on the assessment and no longer see the "waiting for simulators" text)
3. Use the notebook  title

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

