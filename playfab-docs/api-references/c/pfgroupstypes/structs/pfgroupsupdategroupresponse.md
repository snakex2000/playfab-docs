---
author: jasonsandlin
title: "PFGroupsUpdateGroupResponse"
description: "PFGroupsUpdateGroupResponse data model."
ms.author: jasonsa
ms.topic: reference
ms.service: azure-playfab
ms.date: 02/22/2024
---

# PFGroupsUpdateGroupResponse  

PFGroupsUpdateGroupResponse data model.  

## Syntax  
  
```cpp
typedef struct PFGroupsUpdateGroupResponse {  
    const char* operationReason;  
    int32_t profileVersion;  
    PFOperationTypes const* setResult;  
} PFGroupsUpdateGroupResponse;  
```
  
### Members  
  
**`operationReason`** &nbsp; const char*  
*is null-terminated*  
  
(Optional) Optional reason to explain why the operation was the result that it was.
  
**`profileVersion`** &nbsp; int32_t  
  
New version of the group data.
  
**`setResult`** &nbsp; [PFOperationTypes](../../pftypes/enums/pfoperationtypes.md) const*  
*may be nullptr*  
  
(Optional) Indicates which operation was completed, either Created, Updated, Deleted or None.
  
  
## Requirements  
  
**Header:** PFGroupsTypes.h
  
## See also  
[PFGroupsTypes members](../pfgroupstypes_members.md)  

  
  
