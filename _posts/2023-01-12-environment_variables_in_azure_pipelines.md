---
layout: post
title: Environment variables in Azure Pipelines
description: How to add environment variables in azure pipelines for environment dependant cli's
categories: [azure pipelines]
---
When you have a program that fetches values from the environment variables you have to handle this in a certain way in Azure pipelines.
Initially, I though adding a variable to the variables section of the azure pipeline was enough like this: 

```yaml
variables: 
- name: InputEnvironmentVariable
  value: 'value'
```

But, what happends when you do this? I had to check, so created a powershell task that printed all environment variables available. (Fyi, this is a great debugging method if you are trying to debug variables)

```yaml
- task: PowerShell@2
  displayName: Output env variables
  inputs: 
    targetType: inline
    script: (gci  env:* | sort-object name)
```

The output looked like this: 
```
Name                           Value                                                                                   
----                           -----                                                                                   
agent.jobstatus                Succeeded                                                                               
AGENT_BUILDDIRECTORY           c:\vstsBuildAgent\Agent\_work\1                                                     
AGENT_DISABLELOGPLUGIN_TEST... true                                                                                    
AGENT_DISABLELOGPLUGIN_TEST... true
[...]
```

Unfortunetly, I was not able to find InputEnvironmentVariable in the output.
After visiting the task I found that you are able to add environment variable to tasks by using the `env` property on the task.
```yaml
  - task: PowerShell@2
    displayName: Output env variables with one extra
    env: 
      InputEnvironmentVariable: ${{ variables.InputEnvironmentVariable }}
    inputs: 
      targetType: inline
      script: (gci  env:* | sort-object name)
```
The result contained the environment variable!
```
Name                           Value                                                                                   
----                           -----                                                                                   
agent.jobstatus                Succeeded                                                                               
AGENT_BUILDDIRECTORY           c:\vstsBuildAgent\Agent\_work\1                                                     
AGENT_DISABLELOGPLUGIN_TEST... true                                                                                    
AGENT_DISABLELOGPLUGIN_TEST... true
[...]
InputEnvironmentVariable       value
```

Therefore, if you have a exe or a task that is dependant that environment variables exists. Use the `env` property! 
