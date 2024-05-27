
# Research

## Suggested format
{range-predicate}path-selection[constraint-predicate](filter-predicate)

filter-predicate: LEFT_CIRCLE_BRACES PHRASE RIGHT_CIRCLE_BRACES
```
filter-predicate: 
    LEFT_CIRCLE_BRACES
        LEFT_LOGICAL_CIRCLE_BRACES
            LEFT_LOGICAL_CIRCLE_BRACES 
                PHRASE (1...n) with LOGICAL_OPERATOR 
            RIGHT_LOGICAL_CIRCLE_BRACES 
            LOGICAL_OPERATOR 
            PHRASE
        RIGHT_LOGICAL_CIRCLE_BRACES
        LEFT_LOGICAL_CIRCLE_BRACES
            PHRASE (1...n) with LOGICAL_OPERATOR
        RIGHT_LOGICAL_CIRCLE_BRACES
        LOGICAL_OPERATOR
        PHRASE
        ...
    RIGHT_CIRCLE_BRACES
```

LEFT_CIRCLE_BRACES: ( , must

RIGHT_CIRCLE_BRACES: ) , must

LEFT_LOGICAL_CIRCLE_BRACES: (, optional, based on the logical phrase value

RIGHT_LOGICAL_CIRCLE_BRACES: ), optional, based on the logical phrase value

PHRASE: POINTpath-selection[constraint-predicate] OPERATOR VALUE

POINT = internal path selection will always start with **.** ; Means that the relevant  path-selection will start from the select object which is root, selected root object represented as .

OPERATOR: =, !=, >, <, >=, >-, contains, none-contains, match-regex, is null ( similar to '= null'), not null ( similar to '!= null')

VALUE: "string value", TRUE, FALSE, int value, dec(decimal value)

LOGICAL_OPERATOR = &&, ||, optional, based on the logical phrase value

## Text
```javascript
mutation createCompany {
createCompany_CompanySetupInfo(
input: {
    clientMutationId: "1"
    companyCompanySetupInfo: {
    profile: {
    localization: { country: "US" }
    companyName: "Rcc Paid company"
    contactMethods: [
        {
            addresses: [
                {
                    addressComponents: [
                        { name: "CITY", value: "New York" }
                        { name: "cityOrLocality", value: "New York" }
                        { name: "STATE", value: "NY" }
                        { name: "stateOrProvince", value: "NY" }
                        { name: "ZIP", value: "100å38" }
                        { name: "zipcode", value: "10038" }
                        { name: "ADDRESS_LINE_1", value: "2491 Turkey Pen Road" }
                        { name: "address1", value: "2491 Turkey Pen Road" }
                        { name: "COUNTRY", value: "US" }
                        { name: "country", value: "US" }
                    ]
                }
            ]
            emails: [{ emailAddress: "rcctestemail@gmail.com" }]
        }
    ]
    industryType: "ALL_OTHER_MISCELLANEOUS_SERVICES"
    }
}
```

## Use cases
1. Select objects - already  supported, not by filter
2. Select objects by operator to ... input value
3. 2. Select objects by operator to ... input value, check internal path exist and object is not null
3. Select objects by internal values combination by logical operators (&&, ||)


## Thoughts
use case 1:
//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg]

use case 2:
//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.clientMutationId=1)
//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.companyCompanySetupInfo/profile/localization/country != "US)

Select input[type=arg] where internal clientMutationId is 1
//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.clientMutationId=1 && .companyCompanySetupInfo/profile/localization.country != "US)
//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg]( **(**.clientMutationId=1 
    && .companyCompanySetupInfo/profile/localization/country) != "US **)** 
    || .companyCompanySetupInfo/profile/contactMethods/emails contains "gmail.com")


//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.companyCompanySetupInfo/profile/contactMethods/addresses not null)

//{ name: "ADDRESS_LINE_1", value: "2491 Turkey Pen Road" }
here the name is not null and select by its value of 'ADDRESS_LINE_1' but there is also the value that can be select by "2491 Turkey Pen Road", i am thinking how to do it without making it hard to understand and be adopted.

//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.companyCompanySetupInfo/profile/contactMethods/addresses/addressComponents/name == 'ADDRESS_LINE_1')

leads to-->
//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.companyCompanySetupInfo/profile/contactMethods/addresses/addressComponents -> (name == 'ADDRESS_LINE_1' && value != "new york"))

another option is to have canonical path selection which is 

//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.companyCompanySetupInfo/profile/contactMethods/addresses/addressComponents/name == "ADDRESS_LINE_1"(.companyCompanySetupInfo/profile/contactMethods/addresses/addressComponents/value != "new york"))

which leads to:
- too long phrase
- cumbersome
- hard to understand

we need the ability to define a composite condition over the same object, which help to understand the first approache is most correct one