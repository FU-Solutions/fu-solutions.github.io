---
title: Oracle RAC Backup with Backup Exec
author: w3ich3rt
date: 2016-11-13 13:00:00 +0100
categories: [Linux]
tags: [backup, backup exec, linux, oracle, rac, shell]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Zabbix Logo  
---

All configurations needs to be done on every RAC-Node. All shell-outputs are examples.

## Prerequisites

### Oratab and Beoratab
 
The following step are essential on every RAC-Instance.
`cp /etc/oratab /etc/VRTSralus/beoratab`
Setting the permissions for the `beoratab`:

```shell
 chown root:beoper /etc/VRTSralus/beoratab
chmod g+w /etc/VRTSralus/beoratab
```

Editing the settings in the beoratab...

```shell
 -MGMTDB:/opt/grid/12.1.0.2:N            # line added by Agent
+ASM1:/opt/grid/12.1.0.2:N              # line added by Agent
DB1_RAC:/opt/oracle/product/12.1.0.2/dbhome_1:N               # line added by Agent
DB2_RAC:/opt/oracle/product/12.1.0.2/dbhome_1:N            # line added by Agent
DB3_RAC:/opt/oracle/product/12.1.0.2/dbhome_1:N             # line added by Agent
DB4_RAC:/opt/oracle/product/12.1.0.2/dbhome_1:N            # line added by Agent
DB5_RAC:/opt/oracle/product/12.1.0.2/dbhome_1:N              # line added by Agent
```

You have to change the Oracle SID ( *Values before the **":"*** ) to the Oracle SID of the Node you are working on. You can get the Oracle SID with`ps -ef | grep pmon`. The result should look like that:

```shell
 [root@rac01 log]# ps -ef | grep pmon
oracle    7123     1  0 Sep07 ?        00:02:13 asm_pmon_+ASM1
root     13413 31904  0 12:43 pts/6    00:00:00 grep pmon
oracle   14783     1  0 Sep08 ?        00:03:12 ora_pmon_DB1
oracle   19193     1  0 Sep08 ?        00:04:00 ora_pmon_DB2
oracle   19198     1  0 Sep08 ?        00:03:15 ora_pmon_DB3
oracle   19205     1  0 Sep08 ?        00:03:16 ora_pmon_DB4
oracle   19259     1  0 Sep08 ?        00:03:33 ora_pmon_DB15
```

The entries of DB1, DB2, aso. must changed toDB11, DB21, aso..., because we have to use the local instancenames, Backupexec will determine which instance is used by which rac-instance. To make this change you can use `sed`: `sed -i 's/_RAC/1/g' /etc/VRTSralus/beoratab`

> **Info:** here we change the *_RAC* to 1.

### Permissions and Groups

The user `oracle` have to be a member of `beoper`. You can check that by using `id oracle`.

```shell
 id oracle
uid=501(oracle) gid=503(oinstall) groups=503(oinstall),501(asmdba),504(dba),505(oper)
```

Oh s**t the oracle user is not in the beoper group, ok we change that with `usermod`.

```shell
 usermod -aG beoper
id oracle
uid=501(oracle) gid=503(oinstall) groups=503(oinstall),501(asmdba),504(dba),505(oper),506(beoper)
```

## Configure the backup exec agent

Okay the `/etc/VRTSralus/beoratab` is in shape, so we can configure the agent. Before we start, we need some informations: - Passwords of the Oracle-Systemusers - The backupaccount and the specific passwords for the databases + The backupuser needs the `sysbackup` rights on the databases + Also, the oraclepasswordfile of each databaseinstance needs this setting `sysbackup=y` in it. Call your DBA if you do not have the right permissions.
Ok now we can start, we use this little helper `/opt/VRTSralus/bin/AgentConfig` to change the configfile of the backupexec agent. You also can do this via `vi`, but thats not recommended.

> First of all, we need to set the systemcredentials. 
> Do that as root:

```shell
 /opt/VRTSralus/bin/AgentConfig
Symantec Backup Exec Remote Agent Utility
    Choose one of the following options:
    1. Configure database access
    2. Configure Oracle instance information
    3. Quit
    Please enter your selection: 1
Configuring machine information
    Choose one of the following options:
    1. Add system credentials for Oracle operations
    2. Edit system credentials used for Oracle operations
    3. Remove system credentials used for Oracle operations
    4. View system credentials used for Oracle operations
    5. Quit
    Please enter your selection: 1
    Enter a user name that has local system credentials: oracle
    Enter the password:
    Re-enter password:
    Validating credentials…….
    Do you want to use a custom port to connect to the media server during Oracle operations? (Y/N): N
    Commit Oracle operation settings to the configuration file? (Y/N): Y
    SUCCESS: Successfully added the entry to the configuration file.
```

Now we have to leave the tool, to configure the database-instanceinformation. This has to be done in the oracleusercontext. If you dont do that, backup exec maybe cannot authenticate on the databases and the backupjobs will fail.

```shell
 Symantec Backup Exec Remote Agent Utility
    Choose one of the following options:
    1. Configure database access
    2. Configure Oracle instance information
    3. Quit
Note: If the Oracle instance you want to protect is part of an Oracle RAC setup with version 12c, then before selecting the 'Configure Oracle instance information' option, please switch to the Oracle user.
    Please enter your selection: 2
Configuring the Oracle Agent
    Choose one of the following options:
    1. Add a new Oracle instance to protect
    2. Edit an existing Oracle instance
    3. Delete an existing Oracle instance
    4. View Oracle instance entries that have been added in the Remote Agent Utility
    5. Quit
    Please enter your selection: 1
    Select an Oracle instance to configure
        Entry 1. DB1
        Entry 2. DB2
        Entry 3. DB3
        Entry 4. DB4
        Entry 5. DB5
        Enter the number 0 to go back
    Enter your selection: 1
    Enter the Oracle database SYSBACKUP user name: SYSBACKUP
    Enter the Oracle database SYSBACKUP password:
    Re-enter password:
    Validating credentials.......
    Enter the auxiliary instance path for PDB restores:
    Enter the Backup Exec server name or IP address: &lt;IP.DES.BACKUP.SERVERS&gt;
    Do you use a recovery catalog? (Y/N):Y
    Enter the recovery catalog TNSNAME: &lt;CATALOG_TNSNAME&gt;
    Enter the recovery catalog user name: rman_DB1
    Enter the recovery catalog password:
    Re-enter password:
    Validating credentials.......
    Do you want to use a customized job template? (Y/N): N
    Commit Oracle operation settings to the configuration file? (Y/N): Y
    SUCCESS: Successfully added the entry to the configuration file.
```

You have to repeat this step for every databaseinstance, you also can edit the configfile `/etc/VRTSralus/ralus.cfg` with vi and copy, paste and change the values.

## Restart the backupexec agent

When you change something in the `/etc/VRTSralus/ralus.cfg` (The AgentConfig tool also changes the `ralus.cfg`) you have to restart the agent. So lets do this.

```shell
 /etc/init.d/VRTSralus.init restart
## Output from shell
Stopping Symantec Backup Exec Remote Agent ..
Stopping Symantec Backup Exec Remote Agent:                              [  OK  ]
Starting Symantec Backup Exec Remote Agent ......
Starting Symantec Backup Exec Remote Agent:                              [  OK  ]
```

## Configuration in Backup Exec

In Backup Exec have to create the Login-Accounts for each RAC-Node and a virtual RAC-Node. If you have RAC-Nodes, you have to create three entries. Its important that you create a Oracle-Systemuser and a Database-Account. To make that easy, they should be the same users on each RAC-Node.
In the backup and restore menu the RAC-Instances will appear automaticly with names like `RAC-<Databasename>-<Database-ID>`. Now you can backup them with backup exec.
