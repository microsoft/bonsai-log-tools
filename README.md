# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Description

This notebook provides you with a template to import and visualize logs for specific brain versions. The data is imported from an Azure Log Analytics Workspace (LAW) and into Jupyter.

Training and assessment logs are saved in an automatically provisioned [Azure Log Analytics workspace](https://docs.microsoft.com/en-us/azure/azure-monitor/log-query/log-analytics-tutorial) (and can be queried via [KQL](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/tutorial?pivots=azuremonitor) language). By default, one simulator is logged for each brain training and assessment session. To collect logs from more simulators during training, use the Bonsai CLI and refer to the instructions below.

This [documentation](https://docs.microsoft.com/en-us/bonsai/cli/brain/version/start-logging) offers more details on enabling logging via the CLI.

## Prerequisites

1. Install requirements: `pip install -r requirements.txt`
 
## Logging training (with unmanaged sims)

1. Start an unmanaged sim and brain training as you normally would: 
    1. register a sim by launching your sim. For example `python main.py` (or through our partner sims AnyLogic or Simulink)
    1. start brain training `bonsai brain version start-training --name <BRAIN_NAME>`
    1. connect your registered sim to a brain `bonsai simulator connect --simulator-name <SIM_NAME> --brain-name <BRAIN_NAME> --version <VERSION_#> --action Train --concept-name <CONCEPT_NAME>`
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
6. Open the jupyter notebook `Bonsai_log_analysis.ipynb` to perform queries, or query using the log analytics workspace environment.
    - start jupyter: ```jupyter notebook```

## Logging training (with managed sims)
1. start brain training `bonsai brain version start-training --name <BRAIN_NAME>`
2. When you're ready to [start logging](https://docs.microsoft.com/en-us/bonsai/cli/brain/version/start-logging):
`bonsai brain version start-logging -n <brain-name> --version <version-number> -m`
    1.Note: A Log Analytics workspace will be provisioned automatically on Azure if it does not already exist
3. Refer to steps 4-6 in the section above

## Logging custom assessements
1. Start a custom assessment (with a managed or unmanaged sim) as you usually would, using either the CLI or the web UI
2. Wait for your assessment to start running (in the UI, you can tell that your assessment is running when you click on the assessment and no longer see the "waiting for simulators" text)
3. Navigate to the CLI to [start logging](https://docs.microsoft.com/en-us/bonsai/cli/brain/version/start-logging)
    1. For managed simulators use the command `bonsai brain version start-logging -n <brain-name> --version <version-number> -m`
    2. For all unmanaged simulators that you want to log from use the command `bonsai brain version start-logging -n <BRAIN_NAME> --session-id <SESSION_ID>`
4. Refers to steps 4-5 in the section one (logging training with unmanaged sims) 
5. Note that one simulator will always be logged from your custom assessments by default. Use the instructions above if you want to log from more than one simulator for your custom assessment. Alternatively, run your entire assessment on one simulator instance (managed or managed) so that all the data from the assessment is logged.
