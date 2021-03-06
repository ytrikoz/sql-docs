---
title: "Enable or disable remote package management for SQL Server | Microsoft Docs"
ms.custom: ""
ms.date: "02/20/2018"
ms.reviewer: 
ms.suite: sql
ms.prod: machine-learning-services
ms.prod_service: machine-learning-services
ms.component: r
ms.technology: 
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: 6e384893-04da-43f9-b100-bfe99888f085
caps.latest.revision: 7
author: "jeannt"
ms.author: "jeannt"
manager: "cgronlund"
ms.workload: "Inactive"
---
# Enable or disable remote package management for SQL Server
[!INCLUDE[appliesto-ss-xxxx-xxxx-xxx-md-winonly](../../includes/appliesto-ss-xxxx-xxxx-xxx-md-winonly.md)]

This article describes how to enable management of R packages from a remote instance of Machine Learning Server. After the package management feature has been enabled, you can use RevoScaleR commands to install packages on a database from a remote client.

> [!NOTE]
> Currently management of R libraries is supported; support for Python is on the roadmap.

By default, the external package management feature for SQL Server is disabled, even if machine learning features have been installed. You must run a separate script to enable the feature as described in the next section.

## Overview of process and tools

To enable or disable package management, use the command-line utility **RegisterRExt.exe**, which is included with the **RevoScaleR** package.

[Enabling](#bkmk_enable) this feature is a two-step process, requiring a database administrator: you enable package management on the SQL Server instance (once per SQL Server instance), and then enable package management on the SQL database (once per SQL Server database).

[Disabling](#bkmk_disable) the package management feature also requires multipel steps: you remove database-level packages and permissions (once per database), and then remove the roles from the server (once per instance).

## <a name="bkmk_enable"></a> Enable package management

1. Open an elevated command prompt and navigate to the folder containing the utility, RegisterRExt.exe. The default location is `<SQLInstancePath>\R_SERVICES\library\RevoScaleR\rxLibs\x64\RegisterRExe.exe`.

2. Run the following command, providing appropriate arguments for your environment:

    `RegisterRExt.exe /installpkgmgmt [/instance:name] [/user:username] [/password:*|password]`

    This command creates instance-level objects on the SQL Server computer that are required for package management. It also restarts the Launchpad for the instance.

    If you do not specify an instance, the default instance is used. If you do not specify a user, the current security context is used. For example, the following command enables package management on the instance in the path of RegisterRExt.exe, using the credentials of the user who opened the command prompt:

    `REgisterRExt.exe /installpkgmgmt`

3. To add package management to a specific database, run the following command from an elevated command prompt:

    `RegisterRExt.exe /installpkgmgmt /database:databasename [/instance:name] [/user:username] [/password:*|password]`
   
    This command creates some database artifacts, including the following database roles that are used for controlling user permissions: `rpkgs-users`, `rpkgs-private`, and `rpkgs-shared`.

    For example, the following command enables package management on the database, on the instance where RegisterRExt is run. If you do not specify a user, the current security context is used.

    `RegisterRExt.exe /installpkgmgmt /database:TestDB`

4. Repeat the command for each database where packages must be installed.

5. To verify that the new roles have been successfully created, in SQL Server Management Studio, click the database, expand **Security**, and expand **Database Roles**.

    You can also run a query on sys.database_principals such as the following:

    ```SQL
    SELECT pr.principal_id, pr.name, pr.type_desc,   
        pr.authentication_type_desc, pe.state_desc,   
        pe.permission_name, s.name + '.' + o.name AS ObjectName  
    FROM sys.database_principals AS pr  
    JOIN sys.database_permissions AS pe  
        ON pe.grantee_principal_id = pr.principal_id  
    JOIN sys.objects AS o  
        ON pe.major_id = o.object_id  
    JOIN sys.schemas AS s  
        ON o.schema_id = s.schema_id;
    ```

After you have enabled this feature, you can use RevoScaleR function to install or uninstall packages from a remote R client.

## <a name="bkmk_disable"></a> Disable package management

1. From an elevated command prompt, run the RegisterRExt utility again, and disable package management at the database level:

    `RegisterRExt.exe /uninstallpkgmgmt /database:databasename [/instance:name] [/user:username] [/password:*|password]`

    This command removes database objects related to package management from the specified database. It also removes all the packages that were installed from the secured file system location on the SQL Server computer.

2. Repeat this command on each database where package management was used.

3.  (Optional) After all databases have been cleared of packages using the preceding step, run the following command from an elevated command prompt:

    `RegisterRExt.exe /uninstallpkgmgmt [/instance:name] [/user:username] [/password:*|password]`

    This command removes the package management feature from the instance. You might need to manually restart the Launchpad service once more to see changes.

