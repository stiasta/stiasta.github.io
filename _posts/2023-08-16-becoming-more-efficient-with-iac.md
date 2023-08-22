---
layout: post
title: Becoming more effective with Infrastructure As Code [WIP]
description: 
categories: [IaC, Azure]
---

_This is a work in progress_

# What is Infrastructure as Code(IaC)?
You probably already know this, or already have an idea about what it is. Nevertheless, to set the stage, let's start with a definition.

Infrastructure as Code is a way to automate the provisioning of infrastructure. In simple terms, it means that you write code to create and manage your infrastructure. This code ensures that your infrastructure is configured in a consistent way, and that it is reproducible. 

# Benefits of IaC?
## Automatic provisioning
The most obvious benefit of IaC is that it allows you to automate provision of infrastructure in a consistent and repeatable way. This is especially useful when you need to create multiple environments, such as dev, test and production. Also, if you need to create multiple resources of the same type, you can do this consistently and quickly.

## Consistent configuration
Consistency is another great benefit of IaC. You can create templates for your infrastructure, and use these to create it in a consistent way. Let's say you need to create an api, and you need to configure it with a database, storage account and a key vault. If you created this with IaC, you can be sure that no matter how many times you provision it, it will be configured in the same way. 

## Version control
A more subtle benefit of IaC is that you can and should store your IaC code in version control. This way you can track changes to your infrastructure, run code reviews, and even roll back to previous versions if needed. This leads to better quality, and a more stable infrastructure. 



[comment]: <> (example - Talk about when we spent days on finding the problem with logging then igure out that it was one flag in IIS named load user profiles that should have been on. - unable to load certificates from the certificate store. - take screen shots of each step needed to find that setting.)
