---
layout: post
title: ConvertFrom-AppLockerPolicyXml
category: powershell
description: Convert the terrible output from Get-AppLockerPolicy to something worth a damn.
tags: powershell windows
---

Hey-Oh!  Its been a while!  Even though I've been locked at home for the past year, I haven't published any new content in forever.  Well, life has been pretty boring so I haven't come across much worth sharing, and I'm also lazy.  But I do have something today finally worth blerg'n about.  So anyway, away we go!

So, in my not so distant past, I was rolling out Windows AppLocker to our organization, I did get it out to most sites, but other stuff came up, other projects, yada yada yada, and I never got it all the way finished.  Now, its back on the todo list, and one of my peers is working on finishing it up.  Anyway, I am assisting with the KB transfer, and I'm also helping to troubleshoot issues that come up as he moves forward.  Well a few days back I was helping him out, and I was on the users computer, who was having a program being blocked that should not have been.  It was based on a recent exception we had made, so I wanted to verify the new policy was in fact being applied from within the GPO.  So, I started using the AppLocker PowerShell cmdlets.  Long story short, what a disappointment those are.

```powershell
Get-Command -Name "*AppLocker*"
```
```
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        Get-AppLockerFileInformation                       1.0        AppLocker
Function        Get-AppLockerPolicy                                1.0        AppLocker
Function        New-AppLockerPolicy                                1.0        AppLocker
Function        Set-AppLockerPolicy                                1.0        AppLocker
Function        Test-AppLockerPolicy                               1.0        AppLocker
```

Wow, a lot to choose from there right.  Also I think [AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview) came out with Windows 7, SP1 maybe, and these are still v1......anyway.  I was pretty sure the `Get-AppLockerPolicy` cmdlet is going to be the one I want, so I can see exactly what rules are being applied.  So, I did the obvious thing right?  _wink wink_.  I ran `Get-Help` and checked out the Syntax":

```powershell
Get-Help -Name Get-AppLockerPolicy
```
```
SYNTAX
    Get-AppLockerPolicy -Local [-Xml] [<CommonParameters>]

    Get-AppLockerPolicy -Domain -Ldap <string> [-Xml] [<CommonParameters>]

    Get-AppLockerPolicy -Effective [-Xml] [<CommonParameters>]
```

Ok, so it looks like you can either get the Domain or Local policy, or the Effective policy, which would be the combination of both I would guess.  So lets do that and see what is actually being applied.

```powershell
Get-AppLockerPolicy -Effective
```
```
RunspaceId          : a3c075e3-c5c8-4120-8f37-b69d9deb38af
Version             : 1
RuleCollections     : {Microsoft.Security.ApplicationId.PolicyManagement.PolicyModel.FilePublisherRule, ,
                      Microsoft.Security.ApplicationId.PolicyManagement.PolicyModel.FilePublisherRule
                      Microsoft.Security.ApplicationId.PolicyManagement.PolicyModel.FilePublisherRule
                      Microsoft.Security.ApplicationId.PolicyManagement.PolicyModel.FilePublisherRule
                      Microsoft.Security.ApplicationId.PolicyManagement.PolicyModel.FilePublisherRule
                      Microsoft.Security.ApplicationId.PolicyManagement.PolicyModel.FileHashRule…}
RuleCollectionTypes : {Appx, Dll, Exe, Msi…}
```

So wow, thats great info there, looks like all the conditions are in the `RuleCollections` property, lets dig in and see what info is in there.

```powershell
Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections | Select-Object -First 1 -Property *
```
```
Capacity       : 4
Count          : 1
IsFixedSize    : False
IsReadOnly     : False
IsSynchronized : False
SyncRoot       : {Microsoft.Security.ApplicationId.PolicyManagement.PolicyModel.FilePublisherRule}
```

So, nothing, like literally nothing...am I missing something?  Maybe in SyncRoot?

```powershell
Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections | Select-Object -First 1 -ExpandProperty SyncRoot | Select-Object -Property *
```
```
Length
------
    79
```

_Length 79_?  So all this is doing is just giving me a string?  And the _value_ of that string is just the object type name?  Like WTF is going on?  So then I tried diving into that other property, the `RuleCollectionTypes` maybe?  Seems like a long shot for sure:

```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollectionTypes
```
```
Appx
Dll
Exe
Msi
```

JFC!  Where are the RULES!  _Insert Batman meme here...GIVE EM TO ME!_  I guess lets try that `-Xml` parameter, I mean, what other options are there...(see help output above, there are none):

```powershell
Get-AppLockerPolicy -Effective -Xml
```

Not going to paste the out of that here, because its huge.  But its not even true _Xml_, its just a string that has all the Xml tags, so you still have to cast it to Xml yourself:

```powershell
[Xml](Get-AppLockerPolicy -Effective -Xml)
```
```
AppLockerPolicy
---------------
AppLockerPolicy
```

Hey, look at that, an actual true Xml object that can be manipulated like any other Xml document.  So, from what I can tell, the only _real option_ you have here is to just spit out the Xml string to your clipboard and then paste it in a text editor and the use the find function to look for any rule or condition you maybe after. If you are just trying to be quick anyway.  But that still seems like the worst solution ever!  So, I decided to make a PowerShell function that would parse the Xml for you into usable, and more importantly, readable objects.  This also gave me an excuse to spend some time in the IDE, which I jumped on, as I don't get to do that near as much as I used to.

So, the **TLDR**: I created a cmdlet called `ConvertFrom-AppLockerPolicyXml`.  This will take in Xml generated from `Get-AppLockerPolicy` and convert it to usable objects.  It also does a few other things under the hood, such as converting the `UserOrGroupSid` property to the corresponding user or group name, it also bundles all the `Conditions` and `Exceptions` into a single properties:

```powershell
Get-AppLockerPolicy -Effective -Xml | ConvertFrom-AppLockerPolicyXml
```
```
Id              : bef5d95f-da42-4e68-b3bc-06f9e683891f
Name            : All My Stuff
Description     :
Type            : Exe
EnforcementMode : AuditOnly
UserOrGroup     : domain\tomohulk
Action          : Allow
Conditions      : {FilePathCondition}
Exceptions      :
```

And you can dive into the Conditions if you want or need to:

```powershell
Get-AppLockerPolicy -Effective -Xml | ConvertFrom-AppLockerPolicyXml | Select-Object -First 1 -ExpandProperty Conditions
```
```
ConditionType: FilePathCondition
Path: %OSDRIVE%\AllMyStuff\*
```

Mind you these are broad and very fake outputs here, but I think you get the idea.  I did use PowerShell Classes to make it easy to cast to custom objects, in my case, hundreds of rule objects.  So you need at least PowerShell 5.1.  I Did all my development and testing for this in pwsh v7.1.2.  And one other bonus from classes is I like that my _custom_ (casted really) objects have valid descriptive type names:

```powershell
(Get-AppLockerPolicy -Effective -Xml | ConvertFrom-AppLockerPolicyXml)[0].GetType()
```
```
IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     False    AppLockerRule                            System.Object
```

And when I say _casting_ I mean it.  There really isn't much _logic_ to this.  It just iterates and then passes in the Xml nodes to the `AppLockerRule` class, and lets the class definition do all the heavy lifting and format the output.  Here is the only _executing code_ really (`$Xml` being the Xml string from `Get-AppLockerPolicy -Xml`):

```powershell
foreach ($fileType in @("Appx", "Dll", "Exe", "Msi", "Script")) {
    $rules = $Xml.AppLockerPolicy.SelectNodes("//RuleCollection[@Type='$fileType']")
    foreach ($ruleType in @("FilePublisherRule", "FilePathRule", "FileHashRule")) {
        foreach ($rule in @($rules.$ruleType)) {
            [AppLockerRule]::new($rules.EnforcementMode, $rules.Type, $rule)
        }
    }
}
```

Cheers, and enjoy!

{% gist e92ff6e8d85862cb0e8a5b69774e38b3 %}
