# DOORS Integration Scripts [![EPL 2.0](https://img.shields.io/badge/License-EPL%202.0-blue.svg)](https://www.eclipse.org/legal/epl-2.0/)

These scripts were created as a result of the OpenReq project funded by the European Union Horizon 2020 Research and Innovation programme under grant agreement No 732463.

The goal is to export a formal module from IBM DOORS 9.6, so that it can be used and modified by the OpenReq environment, and write the changes back. 

## Overview:
Three scripts are currently provided:
* exportToJSON_withTypeAndDomains.dxl: This script exports a formal module from DOORS and creates a file containing JSON data which can be further used.
* importClassificationsFromCSV.dxl: This script reads attributes from a CSV file and sets a specific attribute in a module.
* importDomainAssignmentsFromCSV.dxl: This script reads attributes from a CSV file and sets a specific attribute in a module.

## Expected Module Schema/Attributes
These script are the outcome of a very specific use case. Therefore the expected attributes and underlying data types are here described in detail:
* "REQ Type": This attribute defines whether a requirement is an actual requirement or just informational text. An actual requirement is a requirement you can fulfill. The underlying data type is an enumeration consisting of two values, DEF and Prose. The number and color are irrelevant.
* "REQ Domain": This attribute defines which technical experts need to look at the requirement. The attribute is a multi-selection enumeration. The underlying data type is called TDomains and consists of one value for one expert. The exported number and color are exported as well, but can be any number and are also irrelevant.

## exportToJSON_withTypeAndDomains.dxl
This script exports a formal module from DOORS and creates a file containing JSON data which can be further used.

### Usage
Since all scripts currently require the same user interactions, please see [Using the Scripts](#using-the-scripts).

### Functionality
The script iterates of every requirement of the current module. It is manually converted to JSON, since DXL does not provide a native implementation. Special characters are therefore escaped "manually" (own implementation which is most likely not complete)! So if your exported JSON data may not be valid, please check manually for invalid characters. If you find other characters that need to be escaped, please see [contribution](#how-to-contribute) to help expanding this list :)

### Target JSON Format
The target JSON schema is different from the OpenReq schema. Two objects are exported, the requirements (reqs) and the so called project definitions (pds). These pds contains all possible values of the attribute REQ Domain. The pds is the exported value of the underlying enum type TDomains and is only set if the TDomains is present in the module.
The schema looks like following:
```javascript
{
	reqs: [
    	{
            "toolId": string, //Requirement's internal identifier
            "text": string, //The requirement's text
            "heading": string, //The requirement's heading
            "level": number, //The indention level
            "reqType": string, //The string value of a single-selection enum field.
            "reqDomains": string //The concatinated values (by \n) of a multi-selection enum field
        },
        ...
    ],
    pds: [
    	{
            "domainName": string, //The name of the enum value
            "number": number, //The underlying number value of the enum value
            "color": number //The assigned color of the enum value
        },
        ...
    ]
}
```

Please note that the attributes reqType and reqDomains are only set if the corresponding attribute REQ Type and REQ Domain respectively is found.

## importClassificationsFromCSV.dxl
This script reads values from a CSV file and sets the REQ Type attribute to the read value.

### Usage
Since all scripts currently require the same user interactions, please see [Using the Scripts](#using-the-scripts).

### Functionality
This script iterates over every requirement in the current module. It takes the next line from the CSV file, reads the provided type and if its other than "NotClassified", it sets the current requirement's REQ Type attribute. The n-th requirement is set to the n-th line's value of the CSV file.

If the ids do not match (first column of CSV and requirement's identifier), the line is skipped but the script is not halted.

### Expected CSV File Format
The CSV file is expected to have the file extension `.classified.csv` and to consist of two columns, separated by a semicolon (`;`). This also implies that semicolon (`;`) may only be used as separator. The first one is the identifier of the requirement and the second one is the value that should be set to the requirement's REQ Type attribute. An example file could look as follows:
```CSV
ID_1;DEF
ID_2;Prose
ID_3;DEF
ID_4;NotClassified
```

Each line contains exatly one entry per requirement. The order is expected to be the same as in the module, so no identifier lookup is done. The first column is simply ignored, since this is sufficient for this use case. But please be aware of it!

Three values are possible as second columns and need to match exactly: `DEF`, `Prose` and `NotClassified`. If a line contains `NotClassified`, the corresponding requirement's REQ Type is not changed.

If you want to adapt the script to fit your needs, please be noted that the CSV domain values need to match exactly the values in the attribute type of REQ Type (TType in this case).

## importDomainAssignmentsFromCSV.dxl
This script reads values from a CSV file and sets the REQ Domain attribute to the read values.

### Functionality
The script iterates over the requirements of the current module and sets the value of the attribute REQ Domain according to the provided values in the CSV. Every given domain is **added** for the requirement's attribute REQ Domain. If no domain is found (empty string, no white space!), no changes are applied to the requirement.

The highlighting of the word "added" means that the domains in the CSV are added to the attribute REQ Domain of a requirement. If a domain has been already set, then it is simply set again (no changes). If the domain has not been set yet, it is set. If a domain has already been set, but is not included in the requirement's CSV line, it is not removed from the requirement!

If the first column does not match the requirement's id, an error is printed and the requirement is skipped. There is no rollback, so if a mismatch occurs at the 100th requirement, the previous 99 requirements are already modified. If the 101th requirement id matches the 101th line's 1st column in the CSV, the values are continued to set (no exit of the script if an mismatch is identified).

Once the script is finished, it prints an information `Successfully updated X requirements.`

### Usage
Since all scripts currently require the same user interactions, please see [Using the Scripts](#using-the-scripts).

### Expected CSV File Format
The CSV file is expected to have the file extension `.assigned.csv` and to consist of two columns, separated by a semicolon (`;`). The first one is the identifier of the requirement and the second one are the values that should be set to the requirement's REQ Domain attribute. An example file could look as follows:
```CSV
ID_1;Domain1
ID_2;
ID_3;Domain1,Domain2
```

Each line contains exatly one entry per requirement. The order is expected to be the same as in the module, so no identifier lookup is done.

The second column contains the domains concatinated by comma (`,`). Please be noted that the CSV domain values need to match exactly the values in the attribute type of REQ Domain (TDomains).

This also implies that the character comma (`,`) or semicolon (`;`) may not be used in any other way, only as separator.



## Using the Scripts
To use a script, it is easiest to:
1. Open the DXL script via a text editor.
2. Copy its content.
3. Open the module in which you want to work with. 
4. Select Tools > Edit DXL ... 
5. Paste the content
6. Press Run
7. Click Browse and a file selection dialog pops up
8. Select a file you want to write to/read from
9. Press Close (Sounds stupid, but this is the actual "Ok button")
10. Let the script run

Depending on the script (import...) it may be necessary to have write permissions on the current module.

## How to contribute
See OpenReq project contribution [Guidlines](https://github.com/OpenReqEU/OpenReq/blob/master/CONTRIBUTING.md "Guidlines").

## License
Free use of this software is granted under the terms of the EPL version 2 (EPL2.0).
