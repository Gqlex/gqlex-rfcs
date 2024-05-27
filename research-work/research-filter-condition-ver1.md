
# Research

## Suggested format
{range-predicate}path-selection[constraint-predicate](filter-predicate)

filter-predicate: LEFT_CIRCLE_BRACES PHRASE RIGHT_CIRCLE_BRACES

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

LEFT_CIRCLE_BRACES: ( , must
RIGHT_CIRCLE_BRACES: ) , must

LEFT_LOGICAL_CIRCLE_BRACES: (, optional, based on the logical phrase value
RIGHT_LOGICAL_CIRCLE_BRACES: ), optional, based on the logical phrase value
PHRASE: .path-selection[constraint-predicate] OPERATOR VALUE

OPERATOR: =, !=, >, <, >=, >-, contains, none-contains, match-regex
VALUE: "string value", TRUE, FALSE, int value, dec(decimal value)
LOGICAL_OPERATOR = &&, ||, optional, based on the logical phrase value

## Text
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

## Use cases
1. Select objects - already  supported, not by filter
2. Select objects by operator to ... input value
3. Select objects by internal values combination by logical operators (&&, ||)


## Thoughts
use case 1:
//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg]

use case 2:
//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.clientMutationId=1)
//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.companyCompanySetupInfo.profile.localization.country != "US)

Select input[type=arg] where internal clientMutationId is 1
//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg](.clientMutationId=1 && .companyCompanySetupInfo.profile.localization.country != "US)
//mutation[name=createCompany]/createCompany_CompanySetupInfo/input[type=arg]( **(**.clientMutationId=1 
    && .companyCompanySetupInfo.profile.localization.country) != "US **)** 
    || .companyCompanySetupInfo.profile.contactMethods.emails contains "gmail.com")



