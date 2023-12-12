# Select works and their relevant marketing material

Select works and their marketing material

## Explanation

Products within a work are listed along with relevant marketing material such as description copy, prizes and subject classifications.

```
query GetWorkDetailsById($workId: String!) {
  work(workSearch: {idEq: $workId}) {
    id
    identifyingDoi
    inHouseWorkReference
    title
    subtitle
    authorshipDescription
    contributorPrettyList
    contributions {
      id
      productIds
      onixContributorRoleCode
      onixContributorRole {description}
      contributor {
        __typename
        name
      }
    }
    editionNumber
    mainBicCode: subjectCodes(schemesIn: [BIC], mainOnly: true) {
      __typename
      code
      description
      main
    }
    mainThema: subjectCodes(schemesIn: [THEMA], mainOnly: true) {
      __typename
      code
      description
      main
      ... on Thema {
        parentCode
      }
    }
    mainBisac: subjectCodes(schemesIn: [BISAC], mainOnly: true) {
      __typename
      code
      description
    }
    marketingTexts {
      onixContentAudienceCodes
      onixContentAudiences {description}
      id
      fromDate
      externalTextNumberOfCharacters
      externalText
      internalTextNumberOfCharacters
      internalText
    }
    marketingTier
    onixEditionTypeCodes
    prizes {
      id
      prizeName
      onixPrizeAchievementCode
      prizeJury
      prizeYear
      prizeStatement
      prizeable {id}
      onixPrizeAchievement {
        __typename
        value
        code
        description
        notes
      }
      productIds
    }
    season
    seriesMemberships {
      series {
        __typename
        name
        subtitle
        id
      }
    }
    similarProducts {
      inHouseEdition {id}
      contributions {
        __typename
        contributor {name}
        onixContributorRole {description}
      }
      authorshipDescription
      id
      isbn {isbn13}
      fullTitle
    }
  }
}
```

Define your work ID variable:

```
{
  "workId": YOUR_WORK_ID_HERE
}
```
