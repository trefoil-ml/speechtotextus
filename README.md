In this article, I will detail the steps to turn a recorded discussion into a text audio file, enrich that transcript with sentiment analyses, and then end with a representation of that discussion in a Power report BI to make an analysis.

To do this, we will use several Azure services including mainly:

- An Azure storage account
- The cognitive service "Speech to text"
- Azure Logic App
- Azure Cosmos DB

Below is a diagram of the solution that we will put in place in Azure


![sparkle](Pictures/901.png)

You can also directly deploy the solution in your Azure subscription by clicking the button bellow:

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ffranmer2%2Fspeechtotextus%2Fmaster%2FARM%2520Files%2FTemplate.json)

Then you can test the solution by dropping the [audio files](https://github.com/franmer2/speechtotextus/tree/master/AudioFile) in the container 'audiofiles' created during the deployment. 


## Prerequisites

- An Azure subscription
- [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) 
  

# Azure services creation
## Create a resource group
We will start by creating a resource group in order to host the various services of our audio file transcription solution.

From [Azure portal](https://portal.azure.com), click on "**Create a resource**"

![sparkle](Pictures/001.png)

 Look for "**Resource group**"

 ![sparkle](Pictures/002.png)


Click on "**Create**"

![sparkle](Pictures/003.png)

In "**Resource group**" field, give a name to your resource group. Click on the button "**Review + Create**"

![sparkle](Pictures/004.png)

In the validation screen, click on the button"**Create**"

![sparkle](Pictures/005.png)

Return to the Azure portal home page. Click on the burger menu at the top left and then on "**Resource** **groups**"

![sparkle](Pictures/006.png)

Click on the resource group created previously

![sparkle](Pictures/007.png)

## Azure storage creation

Once in the resource group, click on the button "**Add**" 

![sparkle](Pictures/008.png)

Find the storage account

![sparkle](Pictures/009.png)

Click on "**Create**"

![sparkle](Pictures/010.png)

Complete the creation of the storage account and click on "**Review** + **create**"

![sparkle](Pictures/011.png)

After checking the information for creating the storage account, click on the button "**Create**"

![sparkle](Pictures/012.png)

## Azure Logic App creation
In the previously created resource group, click the button "**Add**"

![sparkle](Pictures/013.png)

Find "**Logic App**"

![sparkle](Pictures/014.png)

Click on "**Create**"

![sparkle](Pictures/015.png)

Complete the step to create the "Logic App" service, then click on "**Review + create**"

![sparkle](Pictures/016.png)

In the validation window, click on the button "**Create**"

![sparkle](Pictures/017.png)

## "Speech to text" cognitive service creation
In the previously created resource group, click the button "**Add**"

![sparkle](Pictures/018.png)

Find "**Speech**"

![sparkle](Pictures/019.png)

Click on "**Create**"

![sparkle](Pictures/020.png)

Complete the creation form, then click on the button"**Create**"

In the "**Pricing Tier**" field, select "**S0**", because only this tier supports batch transcription:

https://docs.microsoft.com/fr-fr/azure/cognitive-services/speech-service/batch-transcription#prerequisites


![sparkle](Pictures/021.png)

## Azure Cosmos DB account creation

In the previously created resource group, click the button "**Add**"

![sparkle](Pictures/022.png)

Find "**Azure Cosmos DB**"

![sparkle](Pictures/023.png)

Click on "**Create**"

![sparkle](Pictures/024.png)

Fill in the service delivery form. Choose API "**Core (SQL)**".
Click on "**Review + create**"

![sparkle](Pictures/025.png)

After validating the information, click on the button"**Create**"

![sparkle](Pictures/026.png)

In the end, you should have 4 services created in your resource group, as illustrated below:
![sparkle](Pictures/908.png)

# Services configuration
## Storage account
We will now create a container to receive the audio files

From your resource group, click the storage account service

![sparkle](Pictures/027.png)

Click on "**Containers**"

![sparkle](Pictures/028.png)

Click on "**+ Container**". Give a name to your container then click on the button "**Ok**".

![sparkle](Pictures/029.png)

The new container should appear in the list as shown below

![sparkle](Pictures/030.png)


### Shared accees signature creation (SAS Token)

In your storage account, in the left pane, click on "**Shared access signature**"

![sparkle](Pictures/904.png)

Define your key by choosing at least :

- Services
- Resource type
- Allowed permissions
- Expiration date

In our sample, we will check the boxes:

- "**Blob**"
- "**Object**" (Important)
- "**Read**"
- And choose an expiration date

Below a screenshot

![sparkle](Pictures/905.png)

After clicking on the button "**Generate SAS and connection string**", you can copy your key. We will need it a little later.

![sparkle](Pictures/906.png)



## Azure Cosmos DB collection creation

We will now create a collection to receive the text transciption of the audio file

From your resource group, click on Azure Cosmos DB

![sparkle](Pictures/031.png)

Double check you are in "**Overview**" blade, then Click on "**+ Add Container**"

![sparkle](Pictures/032.png)

In "**Add Container**", select "**Create new**" to create a new database. in this article, I choose "**Manual**" for "Throughput" and set value to **400**.

Finally, give a name to your collection and choose a partition key. Here I choose the date as the key, because for my project, we will have many files per day and we will have to make reports over a time range varying from week to month.

![sparkle](Pictures/033.png)

After creating the container you should get a result similar to the screenshot below:

![sparkle](Pictures/034.png)

## Azure Logic Apps configuration

To transcribe the audio files that will be deposited in the storage account, we will use the Batch transcription REST API:

https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/batch-transcription

The steps below will allow you to configure the Azure Logic App service to call the "Speech" cognitive service REST API, as soon as an audio file arrives in the container.

From your resource group, click on the Azure Logic Apps service

![sparkle](Pictures/035.png)

Click on "**Blank Logic App**"

![sparkle](Pictures/036.png)

You will be redirected to the Azure Logic Apps editor

![sparkle](Pictures/037.png)

### Création des paramètres

We will start by creating the different parameters that we will need. In addition, these parameters can be accessed in the ARM templates.

Click on "**Parameters**"

![sparkle](Pictures/200.png)

Click on "**Add parameters**

![sparkle](Pictures/907.png)

Add following parameters:

- cognitiveSecret (one of the cognitive service keys)
- azureRegion (Azure region name (here "**canadacentral**"))
- storageAccountName (storage account name (here "**audiofiledemo**"))
- SASToken (the shared access signature created previously)

Fill in the default values as shown below:

![sparkle](Pictures/201.png)

### Azure Logic App flow creation


In the search area, enter "**Blob**", then select "**When a blob is added or modified**"

![sparkle](Pictures/038.png)

Give a name to your connection channel then select the storage account created at the beginning of this article.

![sparkle](Pictures/039.png)

In "**Container**" field, click on the icon on the right (1) and select the container previously created in this article. Set 5 seconds of interval between each check for the presence of new files in the container.
 
![sparkle](Pictures/040.png)

Click on "**+ New step**" in order to create an additional action.

![sparkle](Pictures/041.png)

In the search box, enter the word "json" then click on "Data Operations"

![sparkle](Pictures/042.png)

Select "**Compose**"

![sparkle](Pictures/043.png)

In the input field, enter the JSON part below, knowing that we will complete the part framed in red later.


```javascript
{
  "description": "La description est optionnelle",
  "locale": "en-US",
  "name": "transcription-batch",
  "properties": {
    "AddSentiment": "True",
    "AddWordLevelTimestamps": "True",
    "ProfanityFilterMode": "Masked",
    "PunctuationMode": "DictatedAndAutomatic"
  },
  "recordingsUrl": 
}
```


![sparkle](Pictures/044.png)

Click to the right of "**recordingUrl**" to bring up the dynamic content window. Click on "**Expression**" and then on the function "**concat**" 

![sparkle](Pictures/045.png)



In the first part of the concatenation, enter the text: **'https://'**
Which will give the first part of our concatenation:
 
 ![sparkle](Pictures/202.png)

Then add a comma and the storage account parameters "**storageAccountName"**. Warning, parameters are under "**Dynamic content**".

![sparkle](Pictures/203.png)

Add "**,'.blob.core.windows.net',**"

![sparkle](Pictures/204.png)


We will add the specific part to the container and to the file by retrieving the information from the previous step. Click on "**Dynamic Content**" then  "**List of Files Path**"

![sparkle](Pictures/205.png)

Finally, we will add the information concerning the shared access signature (SAS token).

Add a comma then the parameter "**SASToken**"

![sparkle](Pictures/206.png)


Then click on "**OK**" (or *Update*).

![sparkle](Pictures/207.png)

So you should get something similar to the screenshot below:

![sparkle](Pictures/050.png)

And the concatenation function script for the "**recordingUrl**" field should look like the one below:

```javascript
concat('https://',parameters('storageAccountName'),'.blob.core.windows.net',triggerBody()?['Path'],parameters('SASToken'))

```

Clic on "**New step**"

![sparkle](Pictures/051.png)

In the search box, enter "http" then click on the "**HTTP**" icon:

![sparkle](Pictures/052.png)

Then select "HTTP"

![sparkle](Pictures/053.png)

We will now send the audio file to the cognitive service.
In the configuration of this step, we will enter the following values for the fields below (we will do the Body part later):

- **Method** : POST
- **URI** : concat('https://',parameters('azureRegion'),'.cris.ai/api/speechtotext/v2.0/Transcriptions')
- **Headers** :
  
|               |                  |
| :------------------------ | -------------------------------: |
| Content-Type              |application/json                  |
| Ocp-Apim-Subscription-Key | *Your cognitive service key parameter*|



*********
*********
As seen above, the key to your cognitive service is in the "**Keys and Endpoint**" section of your cognitive service as illustrated below.

![sparkle](Pictures/054.png)

*********
*********

Click in the "**URI**" field then add in "**Expression**" the script below:

```javascript
concat('https://',parameters('azureRegion'),'.cris.ai/api/speechtotext/v2.0/Transcriptions')
```

Then click "**OK**


![sparkle](Pictures/208.png)

Enter the following values for the part
"**Header**"

|               |                  |
| :------------------------ | -------------------------------: |
| Content-Type              |application/json                  |
| Ocp-Apim-Subscription-Key | *Your cognitive service key parameter*|

So we will get something similar to the screenshot below in our Azure Logic App:

![sparkle](Pictures/209.png)

We will now define the field "**Body**".

Click in the "**Body**" field. In the contextual menu, select "**Dynamic content**", then "**Ouput**" under the "**Compose**" section:

![sparkle](Pictures/210.png)

You can rename your stages by clicking on the 3 small dots, then on "**Rename**". For example, I named this step "*Post to cognitive service*"

![sparkle](Pictures/211.png)



So you should get something similar to the screenshot below.


![sparkle](Pictures/212.png)

We will now retrieve the ID of the transcription and the URL of the audio file returned by the cognitive service
Click on the "**New Step**" button to add the following step:

![sparkle](Pictures/213.png)

In the search field enter "**Parse JSON**" then select "**Parse JSON**"

![sparkle](Pictures/214.png)

Click in the "**Content**" field then on "**Body**" which is in "**Dynamic content**" under "**Post to cognitive service**":

![sparkle](Pictures/215.png)

In "**Schema**" field , copy the script below:

```javascript
{
    "properties": {
        "createdDateTime": {
            "type": "string"
        },
        "id": {
            "type": "string"
        },
        "lastActionDateTime": {
            "type": "string"
        },
        "recordingsUrl": {
            "type": "string"
        },
        "status": {
            "type": "string"
        },
        "statusMessage": {
            "type": "string"
        }
    },
    "type": "object"
}
```
![sparkle](Pictures/216.png)

Click on "**New Step**"

![sparkle](Pictures/217.png)



We will now loop until the cognitive service returns the value "**Succeeded**" to us 

In the search field enter the word "**until**", then select the "**Until**" control

![sparkle](Pictures/058.png)

Click on "**Add an action**"

![sparkle](Pictures/059.png)

Then search for the word "**delay**" and select "**Delay**"

![sparkle](Pictures/060.png)

Set the delay to a few seconds. In this example, I set to 10 seconds. Click on "**Add an action**"

![sparkle](Pictures/061.png)

seach http, then select  "**HTTP**"

![sparkle](Pictures/062.png)

Here we will wait for a response from the cognitive department.
In the "**Method**" field, select "**GET**"

in "**URI**" field, copy the formula below:

```javascript
concat('https://',parameters('azureRegion'),'.cris.ai/api/speechtotext/v2.0/Transcriptions/',body('Parse_transcription_ID_and_result_URL')?['id'])

```

For "**Headers**"
|               |                  |
| :------------------------ | -------------------------------: |
| Content-Type              |application/json                  |
| Ocp-Apim-Subscription-Key | *Your cognitive service key parameter*|


![sparkle](Pictures/063.png)

Even if it is not necessarily necessary, but recommended, it is possible to rename the stages. To the right of "HTTP 2", click on the 3 points then on "**Rename**"


![sparkle](Pictures/064.png)

You can name this step "Get from API", for example. Click on "**Add an action**"

![sparkle](Pictures/065.png)

We will now retrieve the result of the previous step and parse the message. In the search field, enter "Parse JSON", then select "**Parse JSON**"

![sparkle](Pictures/066.png)

Click in the "**Content**" field. Select "**Dynamic content**" then "**Body**" under "**Get from API**"

![sparkle](Pictures/067.png)

In the "**Schema**" field, enter the JSON script below

```javascript
{
    "properties": {
        "id": {
            "type": "string"
        },
        "recordingsUrl": {
            "type": "string"
        },
        "status": {
            "type": "string"
        }
    },
    "type": "object"
}
```
You should therefore obtain a result similar to the one below:

![sparkle](Pictures/068.png)

Return to the level of the "**Until**" loop to define the exit condition. Click in the "**Choose a value**" field. In the context menu, click on "**Dynamic content**" then click on "**status**" under "**Parse JSON**":


![sparkle](Pictures/069.png)

Then set the condition to "**is equal to**" "**Succeeded**"

So you should get something similar to the screenshot below. Click on "**New step**"

![sparkle](Pictures/070.png)


********************

As a precaution, save your work from time to time:

![sparkle](Pictures/900.png)

********************



Search for the operation "**Parse JSON**". Click in the "**Content**" field, then "**Dynamic content**" then on "**Body**" under "**Get from API**"

![sparkle](Pictures/071.png)

In the "**Schema**" field, enter the JSON script below:

```javascript
{
    "properties": {
        "createdDateTime": {
            "type": "string"
        },
        "description": {
            "type": "string"
        },
        "id": {
            "type": "string"
        },
        "lastActionDateTime": {
            "type": "string"
        },
        "locale": {
            "type": "string"
        },
        "name": {
            "type": "string"
        },
        "recordingsUrl": {
            "type": "string"
        },
        "reportFileUrl": {
            "type": "string"
        },
        "resultsUrls": {
            "properties": {
                "channel_0": {
                    "type": "string"
                },
                "channel_1": {
                    "type": "string"
                }
            },
            "type": "object"
        },
        "status": {
            "type": "string"
        },
        "statusMessage": {
            "type": "string"
        }
    },
    "type": "object"
}

```

(To obtain the JSON file below, I did a manual test with Postman. I detail this operation at the end of this article)

You should get something similar to the screenshot below. Click on the "**New step**" button:

![sparkle](Pictures/072.png)

In the search field enter "**Condition**" then click on "**Condition**"

![sparkle](Pictures/073.png)


Click on the button "**Add**" then "**Add row**" to add a second condition.
![sparkle](Pictures/074.png)


Click on "**Choose a value**" then in "**Dynamic content**", select "**channel_0**" under "**Parse JSON 2**". Click on "**OK**".

![sparkle](Pictures/075.png)

Choose "**is not equal to**" for the test, then the expression "**null**" as the value to test.

![sparkle](Pictures/076.png)

Repeat for the second line but for "**Channel_1**". You should get something similar to the screenshot below:

![sparkle](Pictures/077.png)

You can rename this step for clarity. for example I named this step "**Test if channel 0 and 1 is not empty**"

![sparkle](Pictures/079.png)


in "**If true**" part, click on "**Add an action**"

![sparkle](Pictures/078.png)


In the search field enter "**http**", then select "**HTTP**
![sparkle](Pictures/080.png)


In the "**Method**" field, choose "**Get**", then in the "**URI**" field select "**Dynamic content**" then "**channel_0**" under "**Parse JSON 2**"

You can rename this step to something like "*Get channel 0 results*".

Click on "**Add an action**"

![sparkle](Pictures/081.png)


In the search field, enter "**parse json**" and click on "**Parse JSON**"

![sparkle](Pictures/082.png)

In the "**Content**" field, click on "**Dynamic content**" then on "**Body**" under "**Get channel 0 results**

![sparkle](Pictures/083.png)

In the "**Schema**" field, copy the script below:

```javascript
{
    "properties": {
        "AudioFileResults": {
            "items": {
                "properties": {
                    "AudioFileName": {
                        "type": "string"
                    },
                    "AudioFileUrl": {},
                    "AudioLengthInSeconds": {
                        "type": "number"
                    },
                    "CombinedResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Display": {
                                    "type": "string"
                                },
                                "ITN": {
                                    "type": "string"
                                },
                                "Lexical": {
                                    "type": "string"
                                },
                                "MaskedITN": {
                                    "type": "string"
                                }
                            },
                            "required": [
                                "ChannelNumber",
                                "Lexical",
                                "ITN",
                                "MaskedITN",
                                "Display"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    },
                    "SegmentResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Duration": {
                                    "type": "integer"
                                },
                                "DurationInSeconds": {
                                    "type": "number"
                                },
                                "NBest": {
                                    "items": {
                                        "properties": {
                                            "Confidence": {
                                                "type": "number"
                                            },
                                            "Display": {
                                                "type": "string"
                                            },
                                            "ITN": {
                                                "type": "string"
                                            },
                                            "Lexical": {
                                                "type": "string"
                                            },
                                            "MaskedITN": {
                                                "type": "string"
                                            },
                                            "Sentiment": {
                                                "properties": {
                                                    "Negative": {
                                                        "type": "number"
                                                    },
                                                    "Neutral": {
                                                        "type": "number"
                                                    },
                                                    "Positive": {
                                                        "type": "number"
                                                    }
                                                },
                                                "type": "object"
                                            },
                                            "Words": {
                                                "items": {
                                                    "properties": {
                                                        "Confidence": {
                                                            "type": "integer"
                                                        },
                                                        "Duration": {
                                                            "type": "integer"
                                                        },
                                                        "DurationInSeconds": {
                                                            "type": "integer"
                                                        },
                                                        "Offset": {
                                                            "type": "integer"
                                                        },
                                                        "OffsetInSeconds": {
                                                            "type": "number"
                                                        },
                                                        "Word": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "required": [
                                                        "Word",
                                                        "Offset",
                                                        "Duration",
                                                        "OffsetInSeconds",
                                                        "DurationInSeconds",
                                                        "Confidence"
                                                    ],
                                                    "type": "object"
                                                },
                                                "type": "array"
                                            }
                                        },
                                        "required": [
                                            "Confidence",
                                            "Lexical",
                                            "ITN",
                                            "MaskedITN",
                                            "Display",
                                            "Sentiment",
                                            "Words"
                                        ],
                                        "type": "object"
                                    },
                                    "type": "array"
                                },
                                "Offset": {
                                    "type": "integer"
                                },
                                "OffsetInSeconds": {
                                    "type": "number"
                                },
                                "RecognitionStatus": {
                                    "type": "string"
                                },
                                "SpeakerId": {}
                            },
                            "required": [
                                "RecognitionStatus",
                                "ChannelNumber",
                                "SpeakerId",
                                "Offset",
                                "Duration",
                                "OffsetInSeconds",
                                "DurationInSeconds",
                                "NBest"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    }
                },
                "required": [
                    "AudioFileName",
                    "AudioFileUrl",
                    "AudioLengthInSeconds",
                    "CombinedResults",
                    "SegmentResults"
                ],
                "type": "object"
            },
            "type": "array"
        }
    },
    "type": "object"
}
```

You should get a result similar to the screenshot below.

Rename the step with something like "**Parse Channel 0**".

Click on "**Add an action**".

![sparkle](Pictures/084.png)

In the search field enter "**http**" then click on "**HTTP**".

![sparkle](Pictures/085.png)

In the "**Method**" field, choose "**Get**", then in the "**URI**" field select "**Dynamic content**" then "**channel_1**" under " **Parse JSON 2** ".

You can rename this step to something like "*Get channel 1 results*".

Click on "**Add an action**".

![sparkle](Pictures/086.png)

In the search field, enter "**parse json**" then click on  "**Parse JSON**".

![sparkle](Pictures/087.png)

Click in the "**Content**" field, then on "**Dynamic content**". Click on "**Body**" which is under "**Get channel 1 results**".

![sparkle](Pictures/088.png)

In the "** Schema **" field, copy the script below:

```javascript
{
    "properties": {
        "AudioFileResults": {
            "items": {
                "properties": {
                    "AudioFileName": {
                        "type": "string"
                    },
                    "AudioFileUrl": {},
                    "AudioLengthInSeconds": {
                        "type": "number"
                    },
                    "CombinedResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Display": {
                                    "type": "string"
                                },
                                "ITN": {
                                    "type": "string"
                                },
                                "Lexical": {
                                    "type": "string"
                                },
                                "MaskedITN": {
                                    "type": "string"
                                }
                            },
                            "required": [
                                "ChannelNumber",
                                "Lexical",
                                "ITN",
                                "MaskedITN",
                                "Display"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    },
                    "SegmentResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Duration": {
                                    "type": "integer"
                                },
                                "DurationInSeconds": {
                                    "type": "number"
                                },
                                "NBest": {
                                    "items": {
                                        "properties": {
                                            "Confidence": {
                                                "type": "number"
                                            },
                                            "Display": {
                                                "type": "string"
                                            },
                                            "ITN": {
                                                "type": "string"
                                            },
                                            "Lexical": {
                                                "type": "string"
                                            },
                                            "MaskedITN": {
                                                "type": "string"
                                            },
                                            "Sentiment": {
                                                "properties": {
                                                    "Negative": {
                                                        "type": "number"
                                                    },
                                                    "Neutral": {
                                                        "type": "number"
                                                    },
                                                    "Positive": {
                                                        "type": "number"
                                                    }
                                                },
                                                "type": "object"
                                            },
                                            "Words": {
                                                "items": {
                                                    "properties": {
                                                        "Confidence": {
                                                            "type": "integer"
                                                        },
                                                        "Duration": {
                                                            "type": "integer"
                                                        },
                                                        "DurationInSeconds": {
                                                            "type": "integer"
                                                        },
                                                        "Offset": {
                                                            "type": "integer"
                                                        },
                                                        "OffsetInSeconds": {
                                                            "type": "number"
                                                        },
                                                        "Word": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "required": [
                                                        "Word",
                                                        "Offset",
                                                        "Duration",
                                                        "OffsetInSeconds",
                                                        "DurationInSeconds",
                                                        "Confidence"
                                                    ],
                                                    "type": "object"
                                                },
                                                "type": "array"
                                            }
                                        },
                                        "required": [
                                            "Confidence",
                                            "Lexical",
                                            "ITN",
                                            "MaskedITN",
                                            "Display",
                                            "Sentiment",
                                            "Words"
                                        ],
                                        "type": "object"
                                    },
                                    "type": "array"
                                },
                                "Offset": {
                                    "type": "integer"
                                },
                                "OffsetInSeconds": {
                                    "type": "number"
                                },
                                "RecognitionStatus": {
                                    "type": "string"
                                },
                                "SpeakerId": {}
                            },
                            "required": [
                                "RecognitionStatus",
                                "ChannelNumber",
                                "SpeakerId",
                                "Offset",
                                "Duration",
                                "OffsetInSeconds",
                                "DurationInSeconds",
                                "NBest"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    }
                },
                "required": [
                    "AudioFileName",
                    "AudioFileUrl",
                    "AudioLengthInSeconds",
                    "CombinedResults",
                    "SegmentResults"
                ],
                "type": "object"
            },
            "type": "array"
        }
    },
    "type": "object"
}
```

Rename this step to something like "*Parse Channel 1*".

Click on "**Add an action**"

![sparkle](Pictures/089.png)

In the search field, "**compose**", then click "**Compose**".

![sparkle](Pictures/090.png)

Rename this step with something like "*Compose document with channel 0 and 1*".

In the "**Inputs**" field, copy the script below:


```javascript
{
  "AudioDetails": [
    {
      "Channel_0": ,
      "Channel_1": 
    }
  ],
  "AudioLink": "",
  "Date": "",
  "Language": "English",
  "MyDate": "",
  "id": ""
}

```
You should get a result similar to the screenshot below:

![sparkle](Pictures/091.png)

Click to the right of **"Channel_0":** and just before the decimal point. Add "**Body**" which is in "**Dynamic content**" under "**Parse Channel 0**"

![sparkle](Pictures/092.png)

Now click to the right of **"Channel_1":**. Add "**Body**" which is in "**Dynamic content**" under "**Parse Channel 1**"

![sparkle](Pictures/093.png)

Click to the right of **"AudioLink":**, between double quotes, then add "**recordingUrl**" which is in "**Dynamic content**" under "**Parse transcription ID and result URL**"

![sparkle](Pictures/094.png)

Click to the right of **"Date":**, between the double quotes, then add the expression:
```javascript
utcNow()
```

Cliquez sur "**OK**"

![sparkle](Pictures/095.png)

Cliquez à droite de **"MyDate":**, entre les doubles quotes, puis rajoutez l'expression


```javascript
formatDateTime(utcNow(),'yyyy-MM-dd')
```

Click on "**OK**"

![sparkle](Pictures/096.png)

Click to the right of **"id":**, between double quotes, then click on "**id**" found in "**Dynamic content**" under "**Parse transcription ID and result URL**"

![sparkle](Pictures/097.png)

So you should get something similar to the screenshot below:

Ckick on "**Add an action**"

![sparkle](Pictures/098.png)

In the search field, enter "**parse**" then click on "**Parse JSON**"

![sparkle](Pictures/099.png)

Rename the step to something like "*Parse document with channel 0 and 1 and additional information*"

In the "**Content**" field, click on "**Outputs**" found in "**Dynamic content**" under "**Compose document with channel 0 and 1**"


![sparkle](Pictures/100.png)


In the "**Schema**" field, add the script below:

```javascript
{
    "properties": {
        "AudioDetails": {
            "items": {
                "properties": {
                    "Channel_0": {
                        "properties": {
                            "AudioFileResults": {
                                "items": {
                                    "properties": {
                                        "AudioFileName": {
                                            "type": "string"
                                        },
                                        "AudioFileUrl": {},
                                        "SegmentResults": {
                                            "items": {
                                                "properties": {
                                                    "ChannelNumber": {},
                                                    "Duration": {
                                                        "type": "integer"
                                                    },
                                                    "NBest": {
                                                        "items": {
                                                            "properties": {
                                                                "Confidence": {
                                                                    "type": "number"
                                                                },
                                                                "Display": {
                                                                    "type": "string"
                                                                },
                                                                "ITN": {
                                                                    "type": "string"
                                                                },
                                                                "Lexical": {
                                                                    "type": "string"
                                                                },
                                                                "MaskedITN": {
                                                                    "type": "string"
                                                                },
                                                                "Words": {
                                                                    "items": {
                                                                        "properties": {
                                                                            "Duration": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Offset": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Word": {
                                                                                "type": "string"
                                                                            }
                                                                        },
                                                                        "required": [
                                                                            "Word",
                                                                            "Offset",
                                                                            "Duration"
                                                                        ],
                                                                        "type": "object"
                                                                    },
                                                                    "type": "array"
                                                                }
                                                            },
                                                            "required": [
                                                                "Confidence",
                                                                "Lexical",
                                                                "ITN",
                                                                "MaskedITN",
                                                                "Display",
                                                                "Words"
                                                            ],
                                                            "type": "object"
                                                        },
                                                        "type": "array"
                                                    },
                                                    "Offset": {
                                                        "type": "integer"
                                                    },
                                                    "RecognitionStatus": {
                                                        "type": "string"
                                                    },
                                                    "SpeakerId": {}
                                                },
                                                "required": [
                                                    "RecognitionStatus",
                                                    "ChannelNumber",
                                                    "SpeakerId",
                                                    "Offset",
                                                    "Duration",
                                                    "NBest"
                                                ],
                                                "type": "object"
                                            },
                                            "type": "array"
                                        }
                                    },
                                    "required": [
                                        "AudioFileName",
                                        "AudioFileUrl",
                                        "SegmentResults"
                                    ],
                                    "type": "object"
                                },
                                "type": "array"
                            }
                        },
                        "type": "object"
                    },
                    "Channel_1": {
                        "properties": {
                            "AudioFileResults": {
                                "items": {
                                    "properties": {
                                        "AudioFileName": {
                                            "type": "string"
                                        },
                                        "AudioFileUrl": {},
                                        "SegmentResults": {
                                            "items": {
                                                "properties": {
                                                    "ChannelNumber": {},
                                                    "Duration": {
                                                        "type": "integer"
                                                    },
                                                    "NBest": {
                                                        "items": {
                                                            "properties": {
                                                                "Confidence": {
                                                                    "type": "number"
                                                                },
                                                                "Display": {
                                                                    "type": "string"
                                                                },
                                                                "ITN": {
                                                                    "type": "string"
                                                                },
                                                                "Lexical": {
                                                                    "type": "string"
                                                                },
                                                                "MaskedITN": {
                                                                    "type": "string"
                                                                },
                                                                "Words": {
                                                                    "items": {
                                                                        "properties": {
                                                                            "Duration": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Offset": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Word": {
                                                                                "type": "string"
                                                                            }
                                                                        },
                                                                        "required": [
                                                                            "Word",
                                                                            "Offset",
                                                                            "Duration"
                                                                        ],
                                                                        "type": "object"
                                                                    },
                                                                    "type": "array"
                                                                }
                                                            },
                                                            "required": [
                                                                "Confidence",
                                                                "Lexical",
                                                                "ITN",
                                                                "MaskedITN",
                                                                "Display",
                                                                "Words"
                                                            ],
                                                            "type": "object"
                                                        },
                                                        "type": "array"
                                                    },
                                                    "Offset": {
                                                        "type": "integer"
                                                    },
                                                    "RecognitionStatus": {
                                                        "type": "string"
                                                    },
                                                    "SpeakerId": {}
                                                },
                                                "required": [
                                                    "RecognitionStatus",
                                                    "ChannelNumber",
                                                    "SpeakerId",
                                                    "Offset",
                                                    "Duration",
                                                    "NBest"
                                                ],
                                                "type": "object"
                                            },
                                            "type": "array"
                                        }
                                    },
                                    "required": [
                                        "AudioFileName",
                                        "AudioFileUrl",
                                        "SegmentResults"
                                    ],
                                    "type": "object"
                                },
                                "type": "array"
                            }
                        },
                        "type": "object"
                    }
                },
                "required": [
                    "Channel_0",
                    "Channel_1"
                ],
                "type": "object"
            },
            "type": "array"
        },
        "AudioLink": {
            "type": "string"
        },
        "Date": {
            "type": "string"
        },
        "Language": {
            "type": "string"
        },
        "MyDate": {
            "type": "string"
        },
        "id": {
            "type": "string"
        }
    },
    "type": "object"
}
```

You should get something similar to the screenshot below.

Click on "**Add an action**".

![sparkle](Pictures/101.png)


In the search field, enter "**cosmos**", then click on "**Create or update document**"


![sparkle](Pictures/102.png)


*********************************
It is very likely that Azure Logic App will ask you to create a connection to your Azure Cosmos DB account.

Choose your account and click on the "**Create**" button

![sparkle](Pictures/107.png)


*********************************


Complete the "**Database ID**" and "**Collection ID**" fields with your Azure Cosmos DB account information

![sparkle](Pictures/103.png)

Click in the "**Document**" field, then on "**Body**" which is in "**Dynamic content**" under "**Parse document with channel 0 and 1**"


![sparkle](Pictures/104.png)


Click on "**Add new parameter**"


![sparkle](Pictures/105.png)


Then choose "**Partition key value**"


![sparkle](Pictures/106.png)

Then **between double quotes**, add the field "**MyDate**" which is in "**Dynamic content**" under "**Parse document with channel 0 and 1**"


![sparkle](Pictures/108.png)

We will now process audio files with only one channel. As the stages are almost identical to those which we have just seen, I will not detail them.

Below is an overview of the "**If false**" branch of our condition:

![sparkle](Pictures/109.png)

In the "**If false**" part of the first condition "**Test if channel 0 and 1 is not empty**", add a second condition to test only channel 0.

![sparkle](Pictures/218.png)


In the "**If true**" part of our second condition, add the processing of channel 0 only, following the previous steps **but with the following nuances**:

For HTTP step:

![sparkle](Pictures/219.png)

Parse JSON step:

![sparkle](Pictures/220.png)

Copy the script below into the schema field:

```javascript
{
    "properties": {
        "AudioFileResults": {
            "items": {
                "properties": {
                    "AudioFileName": {
                        "type": "string"
                    },
                    "AudioFileUrl": {},
                    "AudioLengthInSeconds": {
                        "type": "number"
                    },
                    "CombinedResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Display": {
                                    "type": "string"
                                },
                                "ITN": {
                                    "type": "string"
                                },
                                "Lexical": {
                                    "type": "string"
                                },
                                "MaskedITN": {
                                    "type": "string"
                                }
                            },
                            "required": [
                                "ChannelNumber",
                                "Lexical",
                                "ITN",
                                "MaskedITN",
                                "Display"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    },
                    "SegmentResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Duration": {
                                    "type": "integer"
                                },
                                "DurationInSeconds": {
                                    "type": "number"
                                },
                                "NBest": {
                                    "items": {
                                        "properties": {
                                            "Confidence": {
                                                "type": "number"
                                            },
                                            "Display": {
                                                "type": "string"
                                            },
                                            "ITN": {
                                                "type": "string"
                                            },
                                            "Lexical": {
                                                "type": "string"
                                            },
                                            "MaskedITN": {
                                                "type": "string"
                                            },
                                            "Sentiment": {
                                                "properties": {
                                                    "Negative": {
                                                        "type": "number"
                                                    },
                                                    "Neutral": {
                                                        "type": "number"
                                                    },
                                                    "Positive": {
                                                        "type": "number"
                                                    }
                                                },
                                                "type": "object"
                                            },
                                            "Words": {
                                                "items": {
                                                    "properties": {
                                                        "Confidence": {
                                                            "type": "integer"
                                                        },
                                                        "Duration": {
                                                            "type": "integer"
                                                        },
                                                        "DurationInSeconds": {
                                                            "type": "integer"
                                                        },
                                                        "Offset": {
                                                            "type": "integer"
                                                        },
                                                        "OffsetInSeconds": {
                                                            "type": "number"
                                                        },
                                                        "Word": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "required": [
                                                        "Word",
                                                        "Offset",
                                                        "Duration",
                                                        "OffsetInSeconds",
                                                        "DurationInSeconds",
                                                        "Confidence"
                                                    ],
                                                    "type": "object"
                                                },
                                                "type": "array"
                                            }
                                        },
                                        "required": [
                                            "Confidence",
                                            "Lexical",
                                            "ITN",
                                            "MaskedITN",
                                            "Display",
                                            "Sentiment",
                                            "Words"
                                        ],
                                        "type": "object"
                                    },
                                    "type": "array"
                                },
                                "Offset": {
                                    "type": "integer"
                                },
                                "OffsetInSeconds": {
                                    "type": "number"
                                },
                                "RecognitionStatus": {
                                    "type": "string"
                                },
                                "SpeakerId": {}
                            },
                            "required": [
                                "RecognitionStatus",
                                "ChannelNumber",
                                "SpeakerId",
                                "Offset",
                                "Duration",
                                "OffsetInSeconds",
                                "DurationInSeconds",
                                "NBest"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    }
                },
                "required": [
                    "AudioFileName",
                    "AudioFileUrl",
                    "AudioLengthInSeconds",
                    "CombinedResults",
                    "SegmentResults"
                ],
                "type": "object"
            },
            "type": "array"
        }
    },
    "type": "object"
}
```



For "**Compose for only channel 0**" step, use the following script:
```javascript
{
  "AudioDetails": [
    {
      "Channel_0":     
    }
  ],
  "AudioLink": "",
  "Date": "",
  "Language": "English",
  "MyDate": "",
  "id": ""
}

```

![sparkle](Pictures/110.png)
 

For the "**Parse document for Channel 0**" step, use the script below:

```javascript
{
    "properties": {
        "AudioDetails": {
            "items": {
                "properties": {
                    "Channel_0": {
                        "properties": {
                            "AudioFileResults": {
                                "items": {
                                    "properties": {
                                        "AudioFileName": {
                                            "type": "string"
                                        },
                                        "AudioFileUrl": {},
                                        "SegmentResults": {
                                            "items": {
                                                "properties": {
                                                    "ChannelNumber": {},
                                                    "Duration": {
                                                        "type": "integer"
                                                    },
                                                    "NBest": {
                                                        "items": {
                                                            "properties": {
                                                                "Confidence": {
                                                                    "type": "number"
                                                                },
                                                                "Display": {
                                                                    "type": "string"
                                                                },
                                                                "ITN": {
                                                                    "type": "string"
                                                                },
                                                                "Lexical": {
                                                                    "type": "string"
                                                                },
                                                                "MaskedITN": {
                                                                    "type": "string"
                                                                },
                                                                "Words": {
                                                                    "items": {
                                                                        "properties": {
                                                                            "Duration": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Offset": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Word": {
                                                                                "type": "string"
                                                                            }
                                                                        },
                                                                        "required": [
                                                                            "Word",
                                                                            "Offset",
                                                                            "Duration"
                                                                        ],
                                                                        "type": "object"
                                                                    },
                                                                    "type": "array"
                                                                }
                                                            },
                                                            "required": [
                                                                "Confidence",
                                                                "Lexical",
                                                                "ITN",
                                                                "MaskedITN",
                                                                "Display",
                                                                "Words"
                                                            ],
                                                            "type": "object"
                                                        },
                                                        "type": "array"
                                                    },
                                                    "Offset": {
                                                        "type": "integer"
                                                    },
                                                    "RecognitionStatus": {
                                                        "type": "string"
                                                    },
                                                    "SpeakerId": {}
                                                },
                                                "required": [
                                                    "RecognitionStatus",
                                                    "ChannelNumber",
                                                    "SpeakerId",
                                                    "Offset",
                                                    "Duration",
                                                    "NBest"
                                                ],
                                                "type": "object"
                                            },
                                            "type": "array"
                                        }
                                    },
                                    "required": [
                                        "AudioFileName",
                                        "AudioFileUrl",
                                        "SegmentResults"
                                    ],
                                    "type": "object"
                                },
                                "type": "array"
                            }
                        },
                        "type": "object"
                    }
                },
                "required": [
                    "Channel_0"
                ],
                "type": "object"
            },
            "type": "array"
        },
        "AudioLink": {
            "type": "string"
        },
        "Date": {
            "type": "string"
        },
        "Language": {
            "type": "string"
        },
        "MyDate": {
            "type": "string"
        },
        "id": {
            "type": "string"
        }
    },
    "type": "object"
}
```
You should get a result similar to the screenshot below:

![sparkle](Pictures/111.png)

Then for the Azure Cosmos DB stage.

![sparkle](Pictures/221.png)


We will now add the last step of our Logic App. This step consists in deleting the transcription information which remains stored at the level of the cognitive service in order to leave room for the next transcription.
Click on the "**New step**" button.


In the search box, enter "**http**" then click on "**HTTP**".

![sparkle](Pictures/121.png)

In the "**Method**" field, select "**DELETE**".
Click in the URI field, then click on "**Expression**". Paste the script below. Click on the button "**OK**".


```javascript
concat('https://',parameters('azureRegion'),'.cris.ai/api/speechtotext/v2.0/Transcriptions/',body('Parse_transcription_ID_and_result_URL')?['id']))
```

![sparkle](Pictures/122.png)



Finally, in the "**header**" field, add the following information:

| Content-Type              |                 application/json |
| :------------------------ | -------------------------------: |
| Ocp-Apim-Subscription-Key | *Your cognitive service key parameter*|

Like shown below:

![sparkle](Pictures/123.png)

Click on "**Save**"

![sparkle](Pictures/124.png)




# Test the solution

Now that we have completed our logic app, we will make sure that our service is well in status "**Enabled**" 


![sparkle](Pictures/129.png)

If not, click the button "**Enable**"

![sparkle](Pictures/130.png)

With Azure Storage Explorer, place the audio file, which is in the [AudioFile](https://github.com/franmer2/speechtotextus/tree/master/AudioFile) folder, into your storage account. Your Azure Logic App must then trigger and start the audio file transcription process

![sparkle](Pictures/131.png)

If all goes well, the operation must proceed successfully

![sparkle](Pictures/132.png)

Click on "**Succeeded**" (or "Failed" depending on the case :)) for more details.

![sparkle](Pictures/133.png)

Check out the result in your Azure Cosmos DB collection.

![sparkle](Pictures/134.png)


# Power BI
Now that the file data is stored in an Azure Cosmos DB database, it's natural to visualize that data via Power BI. I'm not going to detail how to make the report but just illustrate how to get started. And I also [share](https://github.com/franmer2/speechtotextus/tree/master/Power%20BI) with you a report I made, for example.

From Power BI Desktop, click on "**Get Data**" then on "**More**"

![sparkle](Pictures/137.png)

Click on "**Azure**" then on "**Azure Cosmos DB**". Click on "**Connect**"

![sparkle](Pictures/138.png)

Enter the URL of your Azure Cosmos DB account (which you can find on the Azure portal), then click the "**OK**" button

![sparkle](Pictures/139.png)

Enter the key to your Azure Cosmos DB account (which you can find on the Azure portal). Click the "**Connect**" button

![sparkle](Pictures/140.png)

You should be able to navigate your databases and collections. Choose your collection and click the "**Transform Data**" button

![sparkle](Pictures/141.png)

In Power Query Editor, click on the arrows to access the fields in your collection

![sparkle](Pictures/142.png)

Choose the fields you need and click the "**OK**" button

![sparkle](Pictures/143.png)

For columns that still have double arrows, click on them to choose which fields you want to integrate into the report. It is possible to repeat this operation several times, it all depends on the depth of the structure of the document.

![sparkle](Pictures/144.png)

Below is an example of a Power BI report (available under the folder [*Power BI*](https://github.com/franmer2/speechtotextus/tree/master/Power%20BI)). I added the possibility of doing a "**Drillthrough**" to get the details of the conversations.

![sparkle](Pictures/902.png)


To represent the discussion in the report, I used a Gantt Diagram visualization

![sparkle](Pictures/145.png)


# What if I have files in another language?

I had to process files in English and French.
At the time i did this project (summer 2019 :)), I created another container to receive the files in French, I cloned my Logic App, in which I changed the trigger to point it to the new container . Then I changed the configuration of the call to cognitive service to pass the French language as a parameter. With a big restriction! Sentiment analysis is not supported, so you must set this parameter to "** False **" :( !
![sparkle](Pictures/146.png)

The list of supported languages is available [here](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/language-support)

Today, it may be possible to detect the language of the audio file and pass it as a parameter, but I have not looked at this point.


![sparkle](Pictures/151.png)

# Under the hood

You may ask yourself about the long scripts that need to be added during the "Parse JSON" steps and how to generate them. To do this, I used the tool [Postman](https://www.postman.com/) (but I'm not saying it's the best way to do). With Postman, I manually sent an audio file to the cognitive service. With the Postman tool again, I get the results in JSON format.

Below, the Postman's setup to send an audio file to cognitive service.

"**Headers**" configuration

![sparkle](Pictures/125.png)

A "**Body**" configuration example

![sparkle](Pictures/126.png)

We get the result with the "**Get**" command

![sparkle](Pictures/127.png)

I copy the result in the "**Use sample payload to generate schema**" optin, in the "**Parse JSON**" Azure Logic App step.

![sparkle](Pictures/128.png)

# Troubleshooting

One of the errors I had during the tests was that the cognitive service kept the information from the previous audio files. That's why the last stage of the Logic App is a "Delete" action in cognitive service.

In case you have an error with Azure Logic app, you can check with Postman, with a 'GET' command, if information is still present at the cognitive service level. If this is the case, copy the id of the returned document.

![sparkle](Pictures/135.png)


Then use this id to send a "DELETE" command with Postman, in the way shown below:


![sparkle](Pictures/136.png)



