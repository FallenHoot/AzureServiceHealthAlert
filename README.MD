$Location = "norwayeast" #"<replace me>"
$DeploymentName = "ResourceHealth" #"<replace me>"
$AADTenantID = #"<replace me>"
$CentralizedManagementSubscription = "resourcehealth" #"<replace me>"
$CentralizedManagementSubscriptionId = #"<replace me>"
$LogAnalyticsWorkspace = #"<replace me>"
$ResourceGroupName = #"<replace me>"

Connect-AzAccount -Tenant $AADTenantID
Select-AzSubscription -SubscriptionName $CentralizedManagementSubscriptionId

New-AzManagementGroupDeployment -Name "$($DeploymentName)-sub-diag" `
                                -Location $Location `
                                -TemplateFile .\managementGroupTemplates\policyAssignments\DINE-ActivityLogPolicyAssignment.json `
                                -topLevelManagementGroupPrefix $CentralizedManagementSubscription `
                                -logAnalyticsResourceId "/subscriptions/$($CentralizedManagementSubscriptionId)/resourceGroups/$ResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/$LogAnalyticsWorkspace" `
                                -ManagementGroupId $CentralizedManagementSubscription `
                                -Verbose

New-AzSubscriptionDeployment -Name "$($DeploymentName)-la" `
                             -Location $Location `
                             -TemplateFile .\subscriptionTemplates\logAnalyticsWorkspace.json `
                             -rgName "$ResourceGroupName" `
                             -rgRegion "$Location" `
                             -workspaceName "$LogAnalyticsWorkspace" `
                             -workspaceRegion $Location `
                             -retentionInDays "30" `
                             -automationAccountName "aauto-resourcehealth" `
                             -automationRegion $Location `
                             -Verbose


## Doesn't work yet.
New-AzSubscriptionDeployment -Name "$($DeploymentName)-la-alerts" `
                             -alertLocation $Location `
                             -TemplateFile .\alertsTemplates\lLogAlertRules.json `
                             -alertSource "$ResourceGroupName" `
                                Query "$ResourceGroupName" `
                                Resource1 "$ResourceGroupName" `
                                Type "$ResourceGroupName" `
                             -alertSchedule "$Location" `
                                Frequency "$Location" `
                                Time "$Location" `
                             -alertActions "$LogAnalyticsWorkspace" `
                                SeverityLevel "$LogAnalyticsWorkspace" `
                                SuppressTimeinMin "$LogAnalyticsWorkspace" `
                             -actionGrp $Location `
                                ActionGroup $Location `
                                Subject $Location `
                                Webhook $Location `
                             -Verbose


## Example:
# Kusto Query (Logs Query)
AzureActivity
| where CategoryValue == 'ResourceHealth'
| where ActivityStatusValue <> "Resolved"
| extend p = parse_json(Properties)
| evaluate bag_unpack(p, 'Properties_')
| extend ep = parse_json(Properties_eventProperties)
| evaluate bag_unpack(ep, 'EventProperties_' )
| distinct _ResourceId, EventProperties_title, EventProperties_cause
