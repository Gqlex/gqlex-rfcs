
# Research

## Suggested format
{range-predicate}path-selection[constraint-predicate](filter-predicate)

filter-predicate: LEFT_CIRCLE_BRACES CONDITION_PHRASE RIGHT_CIRCLE_BRACES
```
filter-predicate: 
    LEFT_CIRCLE_BRACES
        LEFT_LOGICAL_CIRCLE_BRACES
            LEFT_LOGICAL_CIRCLE_BRACES 
                CONDITION_PHRASE (1...n) with LOGICAL_OPERATOR 
            RIGHT_LOGICAL_CIRCLE_BRACES 
            LOGICAL_OPERATOR 
            CONDITION_PHRASE
        RIGHT_LOGICAL_CIRCLE_BRACES
        LEFT_LOGICAL_CIRCLE_BRACES
            CONDITION_PHRASE (1...n) with LOGICAL_OPERATOR
        RIGHT_LOGICAL_CIRCLE_BRACES
        LOGICAL_OPERATOR
        CONDITION_PHRASE
        ...
    RIGHT_CIRCLE_BRACES
```

LEFT_CIRCLE_BRACES: ( , must

RIGHT_CIRCLE_BRACES: ) , must

LEFT_LOGICAL_CIRCLE_BRACES: (, optional, based on the logical CONDITION_PHRASE value

RIGHT_LOGICAL_CIRCLE_BRACES: ), optional, based on the logical CONDITION_PHRASE value

CONDITION_PHRASE: Combination of SIMPLE_CONDITION_PHRASE and COMPOSITE_CONDITION_PHRASE

SIMPLE_CONDITION_PHRASE: POINTpath-selection[constraint-predicate] OPERATOR VALUE

COMPOSITE_CONDITION_PHRASE: POINTpath-selection[constraint-predicate] -> 
    (KEY_NAME_1 OPERATOR VALUE_1 
        LOGICAL_OPERATOR 
     KEY_NAME_2 OPERATOR VALUE_2 
     ...optional(LOGICAL_OPERATOR ... ))

POINT = internal path selection will always start with **.** ; Means that the relevant  path-selection will start from the select object which is root, selected root object represented as .

OPERATOR: =, !=, >, <, >=, >-, contains, none-contains, match-regex, is null ( similar to '= null'), not null ( similar to '!= null')

VALUE: "string value", TRUE, FALSE, int value, dec(decimal value)

LOGICAL_OPERATOR = &&, ||, optional, based on the logical CONDITION_PHRASE value

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
1. Select objects and value - already  supported, not by filter
2. Select objects by operator to ... input value
3. Select objects by operator to ... input value, check internal path exist and object is not null
4. Select objects by internal values combination by logical operators (&&, ||)


## Thoughts
use case 1:
> //mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg]

select object within a value - select clientMutationId within input argument value
> //mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg].value/clientMutationId

use case 2:
> //mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.clientMutationId=1)
> 
> //mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.companyCompanySetupInfo/profile/localization/country != "US)

Select input[type=arg] where internal clientMutationId is 1
> //mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.clientMutationId=1 && .companyCompanySetupInfo/profile/localization.country != "US)
> 
> //mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg]( **(**.clientMutationId=1 
    && .companyCompanySetupInfo/profile/localization/country) != "US **)** 
    || .companyCompanySetupInfo/profile/contactMethods/emails contains "gmail.com")


> //mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.companyCompanySetupInfo/profile/contactMethods/addresses not null)

---------------
**Compsiste condition**

> //{ name: "ADDRESS_LINE_1", value: "2491 Turkey Pen Road" }

As you can see the name is not null and select by its value of 'ADDRESS_LINE_1' but there is also the value that can be select by "2491 Turkey Pen Road", i am thinking how to do it without making it hard to understand and be adopted.

> //mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.companyCompanySetupInfo/profile/contactMethods/addresses/addressComponents/name == 'ADDRESS_LINE_1')

leads to -->

> //mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.companyCompanySetupInfo/profile/contactMethods/addresses/addressComponents **-> (name == 'ADDRESS_LINE_1' && value != "new york")**)

Another option is to have canonical path selection which is 

> //mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.companyCompanySetupInfo/profile/contactMethods/addresses/addressComponents/name == "ADDRESS_LINE_1"(.companyCompanySetupInfo/profile/contactMethods/addresses/addressComponents/value != "new york"))

Which leads to:
- too long CONDITION_PHRASE
- cumbersome
- hard to understand

we need the ability to define a composite condition over the same object, which help to understand the first approach is most correct one

another example,

Consider we would like to select ```//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg].value/clientMutationId``` based on
contactMethods/addresses ad contactMethods/emails is not null.
in order to optimize the search we should reffer to contactMethods as composite object implies on the filter

The path-selection will be:

> //mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.companyCompanySetupInfo/profile/contactMethods **-> (addresses is not null && emails is not null)**)