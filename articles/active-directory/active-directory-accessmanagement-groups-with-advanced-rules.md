---
title: Populate groups dynamically based on object attributes in Azure Active Directory | Microsoft Docs
description: How-to's to create advanced rules for group membership including supported expression rule operators and parameters.
services: active-directory
documentationcenter: ''
author: curtand
manager: femila
editor: ''

ms.assetid: 04813a42-d40a-48d6-ae96-15b7e5025884
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/02/2017
ms.author: curtand
ms.reviewer: kairaz.contractor
ms.custom: oldportal

---

# Populate groups dynamically based on object attributes
The Azure classic portal provides you with the ability to enable more complex attribute-based dynamic memberships for Azure Active Directory (Azure AD) groups.  

When any attributes of a user or device change, the system evaluates all dynamic group rules in a directory to see if the change would trigger any group adds or removes. If a user or device satisfies a rule on a group, they are added as a member of that group. If they no longer satisfy the rule, they are removed.

> [!NOTE]
> - You can set up a rule for dynamic membership on security groups or Office 365 groups.
>
> - This feature requires an Azure AD Premium P1 license for each user member added to at least one dynamic group. It is not mandatory to actually assign licenses to users for them to be members in dynamic groups, but you do need to have the minimum number of licenses in the tenant to cover all such users. For example: if you have a total of 1,000 unique users in all dynamic groups in your tenant, you need to have at least 1,000 licenses for Azure AD Premium P1, or above, to meet the license requirement.
>
> - You can create a dynamic group for devices or users, but you cannot create a rule that contains both user and device objects.

> - At the moment it is not possible to create a device group based on owning user's attributes. Device membership rules can only reference immediate attributes of device objects in the directory.

## To create an advanced rule
1. In the [Azure classic portal](https://manage.windowsazure.com), select **Active Directory**, and then open your organization’s directory.
2. Select the **Groups** tab, and then open the group you want to edit.
3. Select the **Configure** tab, select the **Advanced rule** option, and then enter the advanced rule into the text box.

## Constructing the body of an advanced rule
The advanced rule that you can create for the dynamic memberships for groups is essentially a binary expression that consists of three parts and results in a true or false outcome. The three parts are:

* Left parameter
* Binary operator
* Right constant

A complete advanced rule looks similar to this: (leftParameter binaryOperator "RightConstant"), where the opening and closing parenthesis are required for the entire binary expression, double quotes are required for the right constant, and the syntax for the left parameter is user.property. An advanced rule can consist of more than one binary expressions separated by the -and, -or, and -not logical operators.
The following are examples of a properly constructed advanced rule:

* (user.department -eq "Sales") -or (user.department -eq "Marketing")
* (user.department -eq "Sales") -and -not (user.jobTitle -contains "SDE")

For the complete list of supported parameters and expression rule operators, see sections below.


Note that the property must be prefixed with the correct object type: user or device.
The below rule will fail the validation:
mail –ne null

The correct rule would be:

user.mail –ne null

The total length of the body of your advanced rule cannot exceed 2048 characters.

> [!NOTE]
> String and regex operations are case insensitive.
> Strings containing quotes " should be escaped using 'character, for example, user.department -eq \`"Sales".
> Only use quotes for string type values, and only use English quotes.
>
>

## Supported expression rule operators
The following table lists all the supported expression rule operators and their syntax to be used in the body of the advanced rule:

| Operator | Syntax |
| --- | --- |
| Not Equals |-ne |
| Equals |-eq |
| Not Starts With |-notStartsWith |
| Starts With |-startsWith |
| Not Contains |-notContains |
| Contains |-contains |
| Not Match |-notMatch |
| Match |-match |
| In | -in |
| Not In | -notIn |

## Operator precedence

All Operators are listed below per precedence from lower to higher, operator in same line are in equal precedence
-any -all
-or
-and
-not
-eq -ne -startsWith -notStartsWith -contains -notContains -match –notMatch -in -notIn

All operators can be used with or without hyphen prefix.

Note that parenthesis are not always needed, you only need to add parenthesis when precedence does not meet your requirements
For example:

   user.department –eq "Marketing" –and user.country –eq "US"

is equivalent to:

   (user.department –eq "Marketing") –and (user.country –eq "US")

## Using the -In and -notIn operators

If you want to compare the value of a user attribute against a number of different values you can use the -In or -notIn operators. Here is an example using the -In operator:

	user.department -In [ "50001", "50002", "50003", “50005”, “50006”, “50007”, “50008”, “50016”, “50020”, “50024”, “50038”, “50039”, “51100” ]

Note the use of the "[" and "]" at the beginning and end of the list of values. This condition evaluates to True of the value of user.department equals one of the values in the list.

## Query error remediation
The following table lists potential errors and how to correct them if they occur

| Query Parse Error | Error Usage | Corrected Usage |
| --- | --- | --- |
| Error: Attribute not supported. |(user.invalidProperty -eq "Value") |(user.department -eq "value")<br/>Property should match one from the [supported properties list](#supported-properties). |
| Error: Operator is not supported on attribute. |(user.accountEnabled -contains true) |(user.accountEnabled -eq true)<br/>Property is of type boolean. Use the supported operators (-eq or -ne) on boolean type from the above list. |
| Error: Query compilation error. |(user.department -eq "Sales") -and (user.department -eq "Marketing")(user.userPrincipalName -match "*@domain.ext") |(user.department -eq "Sales") -and (user.department -eq "Marketing")<br/>Logical operator should match one from the supported properties list above.(user.userPrincipalName -match ".*@domain.ext")or(user.userPrincipalName -match "@domain.ext$")Error in regular expression. |
| Error: Binary expression is not in right format. |(user.department –eq “Sales”) (user.department -eq "Sales")(user.department-eq"Sales") |(user.accountEnabled -eq true) -and (user.userPrincipalName -contains "alias@domain")<br/>Query has multiple errors. Parenthesis not in right place. |
| Error: Unknown error occurred during setting up dynamic memberships. |(user.accountEnabled -eq "True" AND user.userPrincipalName -contains "alias@domain") |(user.accountEnabled -eq true) -and (user.userPrincipalName -contains "alias@domain")<br/>Query has multiple errors. Parenthesis not in right place. |

## Supported properties
The following are all the user properties that you can use in your advanced rule:

### Properties of type boolean
Allowed operators

* -eq
* -ne

| Properties | Allowed values | Usage |
| --- | --- | --- |
| accountEnabled |true false |user.accountEnabled -eq true |
| dirSyncEnabled |true false |user.dirSyncEnabled -eq true |

### Properties of type string
Allowed operators

* -eq
* -ne
* -notStartsWith
* -StartsWith
* -contains
* -notContains
* -match
* -notMatch
* -in
* -notIn

| Properties | Allowed values | Usage |
| --- | --- | --- |
| city |Any string value or $null |(user.city -eq "value") |
| country |Any string value or $null |(user.country -eq "value") |
| companyName | Any string value or $null | (user.companyName -eq "value") |
| department |Any string value or $null |(user.department -eq "value") |
| displayName |Any string value |(user.displayName -eq "value") |
| facsimileTelephoneNumber |Any string value or $null |(user.facsimileTelephoneNumber -eq "value") |
| givenName |Any string value or $null |(user.givenName -eq "value") |
| jobTitle |Any string value or $null |(user.jobTitle -eq "value") |
| mail |Any string value or $null (SMTP address of the user) |(user.mail -eq "value") |
| mailNickName |Any string value (mail alias of the user) |(user.mailNickName -eq "value") |
| mobile |Any string value or $null |(user.mobile -eq "value") |
| objectId |GUID of the user object |(user.objectId -eq "1111111-1111-1111-1111-111111111111") |
| onPremisesSecurityIdentifier | On-premises security identifier (SID) for users who were synchronized from on-premises to the cloud. |(user.onPremisesSecurityIdentifier -eq "S-1-1-11-1111111111-1111111111-1111111111-1111111") |
| passwordPolicies |None DisableStrongPassword DisablePasswordExpiration DisablePasswordExpiration, DisableStrongPassword |(user.passwordPolicies -eq "DisableStrongPassword") |
| physicalDeliveryOfficeName |Any string value or $null |(user.physicalDeliveryOfficeName -eq "value") |
| postalCode |Any string value or $null |(user.postalCode -eq "value") |
| preferredLanguage |ISO 639-1 code |(user.preferredLanguage -eq "en-US") |
| sipProxyAddress |Any string value or $null |(user.sipProxyAddress -eq "value") |
| state |Any string value or $null |(user.state -eq "value") |
| streetAddress |Any string value or $null |(user.streetAddress -eq "value") |
| surname |Any string value or $null |(user.surname -eq "value") |
| telephoneNumber |Any string value or $null |(user.telephoneNumber -eq "value") |
| usageLocation |Two lettered country code |(user.usageLocation -eq "US") |
| userPrincipalName |Any string value |(user.userPrincipalName -eq "alias@domain") |
| userType |member guest $null |(user.userType -eq "Member") |

### Properties of type string collection
Allowed operators

* -contains
* -notContains

| Properties | Allowed values | Usage |
| --- | --- | --- |
| otherMails |Any string value |(user.otherMails -contains "alias@domain") |

| proxyAddresses |SMTP: alias@domain smtp: alias@domain |(user.proxyAddresses -contains "SMTP: alias@domain") |

## Multi-value properties
Allowed operators

* -any (satisfied when at least one item in the collection matches the condition)
* -all (satisfied when all items in the collection match the condition)

| Properties | Values | Usage |
| --- | --- | --- |
| assignedPlans |Each object in the collection exposes the following string properties: capabilityStatus, service, servicePlanId |user.assignedPlans -any (assignedPlan.servicePlanId -eq "efb87545-963c-4e0d-99df-69c6916d9eb0" -and assignedPlan.capabilityStatus -eq "Enabled") |

Multi-value properties are collections of objects of the same type. You can use -any and -all operators to apply a condition to one or all of the items in the collection, respectively. For example:

assignedPlans is a multi-value property that lists all service plans assigned to the user. The below expression will select users who have the Exchange Online (Plan 2) service plan that is also in Enabled state:

```
user.assignedPlans -any (assignedPlan.servicePlanId -eq "efb87545-963c-4e0d-99df-69c6916d9eb0" -and assignedPlan.capabilityStatus -eq "Enabled")
```

(The Guid identifier identifies the Exchange Online (Plan 2) service plan.)

> [!NOTE]
> This is useful if you want to identify all users for whom an Office 365 (or other Microsoft Online Service) capability has been enabled, for example to target them with a certain set of policies.

The following expression will select all users who have any service plan that is associated with the Intune service (identified by service name "SCO"):
```
user.assignedPlans -any (assignedPlan.service -eq "SCO" -and assignedPlan.capabilityStatus -eq "Enabled")
```

## Use of Null values

To specify a null value in a rule, you can use "null" or $null. Example:

   user.mail –ne null
is equivalent to
   user.mail –ne $null

## Extension attributes and custom attributes
Extension attributes and custom attributes are supported in dynamic membership rules.

Extension attributes are synced from on-premises Window Server AD and take the format of "ExtensionAttributeX", where X equals 1 - 15.
An example of a rule that uses an extension attribute would be

(user.extensionAttribute15 -eq "Marketing")

Custom Attributes are synced from on-premises Windows Server AD or from a connected SaaS application and the the format of "user.extension_[GUID]\__[Attribute]", where [GUID] is the unique identifier in AAD for the application that created the attribute in AAD and [Attribute] is the name of the attribute as it was created.
An example of a rule that uses a custom attribute is

user.extension_c272a57b722d4eb29bfe327874ae79cb__OfficeNumber  

The custom attribute name can be found in the directory by querying a user's attribute using Graph Explorer and searching for the attribute name.
Currently we do not support multi-value attributes synchronized from on premises Active Directory.

## "Direct Reports" Rule
You can create a group containing all direct reports of a manager. When the manager's direct reports change in the future, the group's membership will be adjusted automatically.

> [!NOTE]
> 1. For the rule to work, make sure the **Manager ID** property is set correctly on users in your tenant. You can check the current value for a user on their **Profile tab**.
> 2. This rule only supports **direct** reports. It is currently not possible to create a group for a nested hierarchy, e.g. a group that includes direct reports and their reports.

**To configure the group**

1. Follow steps 1-5 from section [To create the advanced rule](#to-create-the-advanced-rule), and select a **Membership type** of **Dynamic User**.
2. On the **Dynamic membership rules** blade, enter the rule with the following syntax:

    *Direct Reports for "{obectID_of_manager}"*

    An example of a valid rule:
```
                    Direct Reports for "62e19b97-8b3d-4d4a-a106-4ce66896a863"
```
    where “62e19b97-8b3d-4d4a-a106-4ce66896a863” is the objectID of the manager. The object ID can be found on manager's **Profile tab**.
3. After saving the rule, all users with the specified Manager ID value will be added to the group.

## Using attributes to create rules for device objects
You can also create a rule that selects device objects for membership in a group. The following device attributes can be used:

| Properties              | Allowed values                     | Usage                                                       |
|-------------------------|------------------------------------|-------------------------------------------------------------|
| accountEnabled          | true false                         | (device.accountEnabled -eq true)                            |
| displayName             | any string value                   | (device.displayName -eq "Rob Iphone”)                       |
| deviceOSType            | any string value                   | (device.deviceOSType -eq "Android")                         |
| deviceOSVersion         | any string value                   | (device.OSVersion -eq "9.1")                                |
| deviceCategory          | a valid device category name       | (device.deviceCategory -eq "BYOD")                          |
| deviceManufacturer      | any string value                   | (device.deviceManufacturer -eq "Samsung")                   |
| deviceModel             | any string value                   | (device.deviceModel -eq "iPad Air”)                         |
| deviceOwnership         | Personal, Company                  | (device.deviceOwnership -eq "Company")                      |
| domainName              | any string value                   | (device.domainName -eq "contoso.com")                       |
| enrollmentProfileName   | Apple Device Enrollment Profile name    | (device.enrollmentProfileName -eq "DEP iPhones")       |
| isRooted                | true false                         | (device.isRooted -eq true)                              |
| managementType          | “MDM” for mobile devices, “PC” for computers managed through the Intune PC agent    | (device.managementType -eq "MDM")                  |
| organizationalUnit      | any string value matching the name of the OU set by on-premises Active Directory | (device.organizationalUnit -eq "US PCs")      |
| deviceId                | a valid Intune deviceId                | (device.deviceId -eq "d4fe7726-5966-431c-b3b8-cddc8fdb717d") |
| objectId                | a valid Azure AD objectId            | (device.objectId -eq "76ad43c9-32c5-45e8-a272-7b58b58f596d") |

> [!NOTE]
> Device rules cannot be created using the "simple rule" dropdown in the Azure classic portal.
>
>

## Changing dynamic membership to static, and vice versa
It is possible to change how membership is managed in a group. This is useful when you want to keep the same group name and ID in the system, so any existing references to the group are still valid; creating a new group would require updating those references.

> [!WARNING]
> When changing an existing static group to a dynamic group, all existing members will be removed from the group, and then the membership rule will be processed to add new members. If the group is used to control access to apps or resources, the original members may lose access until the membership rule is fully processed.
>
> It is a recommended practice to test the new membership rule beforehand to make sure that the new membership in the group is as expected.

1. In the [Azure classic portal](https://manage.windowsazure.com), open the group.
2. Select the **Configure** tab to view the current state of dynamic membership.
3. To make a group static, simply toggle the **Enable dynamic memberships** setting to **NO**. Click the **Save** button in the toolbar below to confirm. Existing members will be kept in the group and from now on the membership rule will not be processed.
4. To make a group dynamic, toggle the setting to **YES**, specify the desired membership rule and click **Save**. Existing members will be removed and the new rule will start processing to add new members.

## Next steps
These articles provide additional information on Azure Active Directory.

* [Troubleshooting dynamic memberships for groups](active-directory-accessmanagement-troubleshooting.md)
* [Managing access to resources with Azure Active Directory groups](active-directory-manage-groups.md)
* [Azure Active Directory cmdlets for configuring group settings](active-directory-accessmanagement-groups-settings-cmdlets.md)
* [Article Index for Application Management in Azure Active Directory](active-directory-apps-index.md)
* [Integrating your on-premises identities with Azure Active Directory](active-directory-aadconnect.md)
