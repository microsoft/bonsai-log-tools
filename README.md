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

Logging simulators with the Bonsai service is conducted through the [Bonsai CLI](https://pypi.org/project/bonsai-cli/) and sent to an automatically provisioned [Azure Log Analytics workspace](https://docs.microsoft.com/en-us/azure/azure-monitor/log-query/log-analytics-tutorial) can be queried via [KQL](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/tutorial?pivots=azuremonitor) language.

For managed simulators, starting logging of your simulators is as simple as:

```bash
bonsai brain version start-logging -n <brain-name> --version <version-number> -m
```

For unmanaged simulators, logging is user-enabled per specific sim's session-id. A sample to query and visualize data in a jupyter notebook is provided in this repository.

## Prerequisites

1. Install requirements: `pip install -r requirements.txt`
1. Temporary manual step: If your azure subscription has not yet been registered to allow Log Analytics workspace resource provider, it needs to be registered.
    1. Determine if registering is required. <SUBCRIPTION_ID> can be found on preview.bons.ai by clicking on id/Workspace info. 
    ```
    az provider show --namespace 'Microsoft.OperationalInsights' -o table --subscription <SUBCRIPTION_ID>
    ```
    2.  If the registrationState is `Registered`, you can skip this step. If not registered, we will need to register it. This is a one-time step per subscription and user will need owner-level permission. If you don't have the appropriate permission, work with your IT admin to execute that step.

    ```
    az login
    az provider register --namespace 'Microsoft.OperationalInsights' --subscription <SUBCRIPTION_ID>
    ```
 
## Usage

1. Start an unmanaged sim and brain training as you normally would: 
    1. register a sim by launching your sim. For example `python main.py` (or through our partner sims AnyLogic or Simulink)
    1. start brain training `bonsai brain version start-training --name <BRAIN_NAME>`
    1. connect your registered sim to a brain `bonsai simulator connect --simulator-name <SIM_NAME> --brain-name <BRAIN_NAME> --version <VERSION_#> --action Train --concept-name <CONCEPT_NAME>`
2. Find the session-id of un-managed sim using Bonsai CLI: `bonsai simulator unmanaged list`
3. When you're ready to start logging:
`bonsai brain version start-logging -n <BRAIN_NAME> --session-id <SESSION_ID>`
    1.Note: A Log Analytics workspace will be provisioned automatically on Azure if it does not already exist
3. You can find the Log Analytics workspace id on portal.azure.com. It will be created under your provisioned resource group `bonsai-rg-<BONSAI WORKSPACE NAME >-<WORKSPACE-ID>`
    1. Note: It might take ~5 minutes to show up if it is the first time using logging feature as the log analytics workspace gets created.
4. Logs will start populating the Log Analytics workspace 3-5 minutes after starting logging
    1. Note: if this is the first time you're using logging you may need to wait for the first episode to finish so that episode-level (EpisodeLog_CL table) logs gets created and filled with at least 1 row of data.
    1. Optional: Navigate to https://ms.portal.azure.com/#blade/Microsoft_Azure_Monitoring_Logs/LogsBlade to query logs in the webUI. Sample KQL query, take 10 samples of IterationLog_CL table for the corresponding sim's session-id:
    ```KQL
    IterationLog_CL
    | where SessionId_s == <SESSION_ID>
    | take 10
    ```
5. Open the jupyter notebook `Bonsai_log_analysis.ipynb` to perform queries, or query using the log analytics workspace environment.
    - start jupyter: ```jupyter notebook```