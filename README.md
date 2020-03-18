In this article, I will detail the steps to turn a recorded discussion into a text audio file, enrich that transcript with sentiment analyses, and then end with a representation of that discussion in a Power report BI to make an analysis.

To do this, we will use several Azure services including mainly:

- An Azure storage account
- The cognitive service "Speech to text"
- Azure Logic App
- Azure Cosmos DB

Below is a diagram of the solution that we will put in place in Azure


![sparkle](Pictures/901.png)

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

In the first part of the concatenation, enter the address of your storage account. This address can be found in the properties of your storage account. **Don't forget the quotes to enter the address: 'https://audiofiledemo.blob.core......'**
 
![sparkle](Pictures/046.png)

What will give the first part of our concatenation:

![sparkle](Pictures/047.png)

Add a comma at the end of the text. We will add the specific part to the container and file by recovering the information from the previous step. Click on "**Dynamic Content**" then on "**List of Files Path**"

![sparkle](Pictures/048.png)

Then click the button "**OK**"

![sparkle](Pictures/049.png)

So you need to get something similar to the screen copy below:

![sparkle](Pictures/050.png)

And the script for the concatenation function should look like the one below:

```javascript
concat('https://audiofiledemo.blob.core.windows.net',triggerBody()?['Path'])

```

Click on "**New step**"

![sparkle](Pictures/051.png)

in the search area, enter "http" and then click on the icon "**HTTP**"

![sparkle](Pictures/052.png)

Select "HTTP"

![sparkle](Pictures/053.png)

We will now send the audio file to the cognitive service.
In the configuration of this step, we will enter the following values for the fields below (we'll do the Body part later):

- **Method** : POST
- **URI** : https://canadacentral.cris.ai/api/speechtotext/v2.0/Transcriptions
- **Headers** :
  
| Content-Type              |                 application/json |
| :------------------------ | -------------------------------: |
| Ocp-Apim-Subscription-Key | *la clef de votre service cognitif*|

Your cognitive service key can be found in the section "**Keys and Endpoint**" of your cognitive service as illustrated below

![sparkle](Pictures/054.png)

So we're going to get at the level of our Azure Logic App, something similar to the screen copy below:

![sparkle](Pictures/055.png)

We will now define the field "**Body**".

Click in the field of "**Bod**y." In the pop-up menu, select "**Dynamic content**" and then "**Ouput**" under the part "**Compose**"


![sparkle](Pictures/056.png)

So you should get something similar to the screen copy below. Click the "**New Step**" button to add the next step:


![sparkle](Pictures/057.png)

We will now loop until the cognitive service returns us the value "**Succeeded**"

Enter the word "**until**" in the search field, then select "**Until**"

![sparkle](Pictures/058.png)

Click on "**Add an action**"

![sparkle](Pictures/059.png)

Then search for the word "**Delay**" and select "**Delay**"

![sparkle](Pictures/060.png)

Set the time limit for a few seconds. In this example, I set over 10 seconds. Click on "**Add an action**"

![sparkle](Pictures/061.png)

Search for http, then select "**HTTP**"

![sparkle](Pictures/062.png)

In the field "**Method**", select "**GET**" and then fill in the URI and Headers fields as before. Here we will wait for a response from the cognitive service.

![sparkle](Pictures/063.png)

It is possible to rename the steps. To the right of "**HTTP 2**", click on the 3 points and then on "**Rename**"


![sparkle](Pictures/064.png)

You can name this step "**Get from API**" . Click "**Add an action**"

![sparkle](Pictures/065.png)

We will now retrieve the result from the previous step and parse the message. In the search field, enter "Parse JSON" and then select "**Parse JSON**"

![sparkle](Pictures/066.png)

Click "**Content**". Select "**Dynamic content**" then "**Body**" under "**Get from API**" 

![sparkle](Pictures/067.png)

In "**Schema**", enter the JSON script below

```javascript
{
    "items": {
        "properties": {
            "status": {
                "type": "string"
            }
        },
        "required": [
            "status"
        ],
        "type": "object"
    },
    "type": "array"
}
```
So you need to get a result similar to the one below

![sparkle](Pictures/068.png)

Go back to the "**Until**" loop level to set the exit condition. Click in the field "**Choose a value**". In the pop-up menu, click "**Expression**" and enter the command below and click the "**OK**" button:



```javascript
body('Parse_JSON')[0]['status']
```
![sparkle](Pictures/069.png)

Then set the condition to "**is equal to**" "**Succeeded**"

============================================

Save your work from time to time:

![sparkle](Pictures/900.png)

============================================

So you need to get something similar to the screen copy below.Click "**New step**"

![sparkle](Pictures/070.png)

Find "**Parse JSON**". Click on "**Content**", then "**Dynamic content**" and then "**Body**" under "**Get from API**"

![sparkle](Pictures/071.png)

In "**Schema**", enter the JSON script below:

```javascript
{
    "items": {
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
            "models": {
                "items": {
                    "properties": {
                        "createdDateTime": {
                            "type": "string"
                        },
                        "datasets": {
                            "type": "array"
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
                        "modelKind": {
                            "type": "string"
                        },
                        "name": {
                            "type": "string"
                        },
                        "properties": {
                            "properties": {
                                "ModelClass": {
                                    "type": "string"
                                },
                                "Purpose": {
                                    "type": "string"
                                },
                                "VadKind": {
                                    "type": "string"
                                }
                            },
                            "type": "object"
                        },
                        "status": {
                            "type": "string"
                        }
                    },
                    "required": [
                        "modelKind",
                        "datasets",
                        "id",
                        "createdDateTime",
                        "lastActionDateTime",
                        "status",
                        "locale",
                        "name",
                        "description",
                        "properties"
                    ],
                    "type": "object"
                },
                "type": "array"
            },
            "name": {
                "type": "string"
            },
            "properties": {
                "properties": {
                    "AddWordLevelTimestamps": {
                        "type": "string"
                    },
                    "Duration": {
                        "type": "string"
                    },
                    "ProfanityFilterMode": {
                        "type": "string"
                    },
                    "PunctuationMode": {
                        "type": "string"
                    }
                },
                "type": "object"
            },
            "recordingsUrl": {
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
        "required": [
            "recordingsUrl",
            "resultsUrls",
            "models",
            "statusMessage",
            "id",
            "createdDateTime",
            "lastActionDateTime",
            "status",
            "locale",
            "name",
            "description",
            "properties"
        ],
        "type": "object"
    },
    "type": "array"
}

```

(To get the JSON file below, I did a manual test with Postman. I detail this operation at the end of this article)

You should get something similar to the screen copy below. Click the button "**New step**" :

![springle](Pictures/072.png)

In the search field enter "**Condition**" and then click "**Condition**"

![springle](Pictures/073.png)

Click in the field "**Choose a value**". In "**Dynamic content**", click on "**channel_0**" under "**Parse JSON 2**"

![springle](Pictures/074.png)

The step is then included in a "**For each**" loop as shown below:

![springle](Pictures/075.png)

Click again on the step "**Condition**". Choose "**Is not equal to**". Click on "**Choose a value**" then on "**Expression**". in "**Fx**" enter "**null**". Click on"**OK**"


![springle](Pictures/076.png)

In "**If true**" section, Click on "**Add an action**"

![springle](Pictures/077.png)

In the search zone, enter "**http**" then select "**HTTP**"

![springle](Pictures/078.png)

With the following action, we will recover the information about channel number 0. To do this, we need the URL containing this information. This URL can be found in the output of the action "*Parse JSON 2*"

In "**Method**" field, select "**GET**". Click in "**URI**", then in "**Dynamic content**" select "**channel_0**" under "**Parse JSON 2**".


![springle](Pictures/079.png)

Rename "**HTTP 2**" by something like "**Get channel 0 results**"

![springle](Pictures/080.png)

Click on "**Add an action**"

![springle](Pictures/081.png)

In the search area enter "**Parse JSON**", then select "**Parse JSON**".

![springle](Pictures/082.png)

Click in the field "**Content**", then "**Dynamic content**", Click on "**Body**" under "**Get channel 0 results**"

![springle](Pictures/083.png)

in "**Schema**", paste the script below:


```javascript
{
    "properties": {
        "properties": {
            "properties": {
                "AudioFileResults": {
                    "properties": {
                        "items": {
                            "properties": {
                                "properties": {
                                    "properties": {
                                        "AudioFileName": {
                                            "properties": {
                                                "type": {
                                                    "type": "string"
                                                }
                                            },
                                            "type": "object"
                                        },
                                        "AudioFileUrl": {
                                            "properties": {},
                                            "type": "object"
                                        },
                                        "SegmentResults": {
                                            "properties": {
                                                "items": {
                                                    "properties": {
                                                        "properties": {
                                                            "properties": {
                                                                "ChannelNumber": {
                                                                    "properties": {},
                                                                    "type": "object"
                                                                },
                                                                "Duration": {
                                                                    "properties": {
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "NBest": {
                                                                    "properties": {
                                                                        "items": {
                                                                            "properties": {
                                                                                "properties": {
                                                                                    "properties": {
                                                                                        "Confidence": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Display": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "ITN": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Lexical": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "MaskedITN": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Sentiment": {
                                                                                            "properties": {
                                                                                                "properties": {
                                                                                                    "properties": {
                                                                                                        "Negative": {
                                                                                                            "properties": {
                                                                                                                "type": {
                                                                                                                    "type": "string"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        },
                                                                                                        "Neutral": {
                                                                                                            "properties": {
                                                                                                                "type": {
                                                                                                                    "type": "string"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        },
                                                                                                        "Positive": {
                                                                                                            "properties": {
                                                                                                                "type": {
                                                                                                                    "type": "string"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        }
                                                                                                    },
                                                                                                    "type": "object"
                                                                                                },
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Words": {
                                                                                            "properties": {
                                                                                                "items": {
                                                                                                    "properties": {
                                                                                                        "properties": {
                                                                                                            "properties": {
                                                                                                                "Duration": {
                                                                                                                    "properties": {
                                                                                                                        "type": {
                                                                                                                            "type": "string"
                                                                                                                        }
                                                                                                                    },
                                                                                                                    "type": "object"
                                                                                                                },
                                                                                                                "Offset": {
                                                                                                                    "properties": {
                                                                                                                        "type": {
                                                                                                                            "type": "string"
                                                                                                                        }
                                                                                                                    },
                                                                                                                    "type": "object"
                                                                                                                },
                                                                                                                "Word": {
                                                                                                                    "properties": {
                                                                                                                        "type": {
                                                                                                                            "type": "string"
                                                                                                                        }
                                                                                                                    },
                                                                                                                    "type": "object"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        },
                                                                                                        "required": {
                                                                                                            "items": {
                                                                                                                "type": "string"
                                                                                                            },
                                                                                                            "type": "array"
                                                                                                        },
                                                                                                        "type": {
                                                                                                            "type": "string"
                                                                                                        }
                                                                                                    },
                                                                                                    "type": "object"
                                                                                                },
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        }
                                                                                    },
                                                                                    "type": "object"
                                                                                },
                                                                                "required": {
                                                                                    "items": {
                                                                                        "type": "string"
                                                                                    },
                                                                                    "type": "array"
                                                                                },
                                                                                "type": {
                                                                                    "type": "string"
                                                                                }
                                                                            },
                                                                            "type": "object"
                                                                        },
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "Offset": {
                                                                    "properties": {
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "RecognitionStatus": {
                                                                    "properties": {
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "SpeakerId": {
                                                                    "properties": {},
                                                                    "type": "object"
                                                                }
                                                            },
                                                            "type": "object"
                                                        },
                                                        "required": {
                                                            "items": {
                                                                "type": "string"
                                                            },
                                                            "type": "array"
                                                        },
                                                        "type": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "type": "object"
                                                },
                                                "type": {
                                                    "type": "string"
                                                }
                                            },
                                            "type": "object"
                                        }
                                    },
                                    "type": "object"
                                },
                                "required": {
                                    "items": {
                                        "type": "string"
                                    },
                                    "type": "array"
                                },
                                "type": {
                                    "type": "string"
                                }
                            },
                            "type": "object"
                        },
                        "type": {
                            "type": "string"
                        }
                    },
                    "type": "object"
                }
            },
            "type": "object"
        },
        "type": {
            "type": "string"
        }
    },
    "type": "object"
}

```
Rename the action into something like "**Parse channel 0 results**".

In "**Schema**", you should have something like illustrated in the screenshot below.

Also rename the first condition by "**Test channel 0**".

Click "**Add an action**" (**ATTENTION** !! Click on the one '**outside**' of the test "*If true*")

![springle](Pictures/084.png)



In the search field, enter the word "**if**", then select "**Condition**".

![springle](Pictures/085.png)

Click on "**Choose a value**". Click on "**Dynamic content**", then on "**channel_1**" under "**Parse JSON 2**"

![springle](Pictures/086.png)

Select "**is not equal to**" and Click on "**Choose a value**" and "**Expression**"
in"**Fx**" field, enter "**null**". Click on "**OK**".


![springle](Pictures/087.png)

In "**If true**" condition part, Click on "**Add an action**"

![springle](Pictures/088.png)


With the following action, we will recover the information about channel number 1. To do this, we need the URL containing this information. This URL can be found in the output of the action "*Parse JSON 2*"

In the search area, enter "**http**" and then select "**HTTP**".

![springle](Pictures/089.png)

In "**Method**" field, select "**GET**". Click "**URI**", then Click "**Dynamic content**" and select "**channel_1**" under "**Parse JSON 2**".

Click on "**Add an action**"


![springle](Pictures/090.png)

In the search field, enter "**Parse JSON**" and then click "**Parse JSON**"


![springle](Pictures/091.png)

Click on "**Content**" field, then "**Dynamic content**" and select "**Body**" under "**Get channel 1 results**"

![springle](Pictures/092.png)

In "**Schema**" field, paste the JSON script below: 

```javascript

{
    "properties": {
        "properties": {
            "properties": {
                "AudioFileResults": {
                    "properties": {
                        "items": {
                            "properties": {
                                "properties": {
                                    "properties": {
                                        "AudioFileName": {
                                            "properties": {
                                                "type": {
                                                    "type": "string"
                                                }
                                            },
                                            "type": "object"
                                        },
                                        "AudioFileUrl": {
                                            "properties": {},
                                            "type": "object"
                                        },
                                        "SegmentResults": {
                                            "properties": {
                                                "items": {
                                                    "properties": {
                                                        "properties": {
                                                            "properties": {
                                                                "ChannelNumber": {
                                                                    "properties": {},
                                                                    "type": "object"
                                                                },
                                                                "Duration": {
                                                                    "properties": {
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "NBest": {
                                                                    "properties": {
                                                                        "items": {
                                                                            "properties": {
                                                                                "properties": {
                                                                                    "properties": {
                                                                                        "Confidence": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Display": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "ITN": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Lexical": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "MaskedITN": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Sentiment": {
                                                                                            "properties": {
                                                                                                "properties": {
                                                                                                    "properties": {
                                                                                                        "Negative": {
                                                                                                            "properties": {
                                                                                                                "type": {
                                                                                                                    "type": "string"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        },
                                                                                                        "Neutral": {
                                                                                                            "properties": {
                                                                                                                "type": {
                                                                                                                    "type": "string"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        },
                                                                                                        "Positive": {
                                                                                                            "properties": {
                                                                                                                "type": {
                                                                                                                    "type": "string"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        }
                                                                                                    },
                                                                                                    "type": "object"
                                                                                                },
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Words": {
                                                                                            "properties": {
                                                                                                "items": {
                                                                                                    "properties": {
                                                                                                        "properties": {
                                                                                                            "properties": {
                                                                                                                "Duration": {
                                                                                                                    "properties": {
                                                                                                                        "type": {
                                                                                                                            "type": "string"
                                                                                                                        }
                                                                                                                    },
                                                                                                                    "type": "object"
                                                                                                                },
                                                                                                                "Offset": {
                                                                                                                    "properties": {
                                                                                                                        "type": {
                                                                                                                            "type": "string"
                                                                                                                        }
                                                                                                                    },
                                                                                                                    "type": "object"
                                                                                                                },
                                                                                                                "Word": {
                                                                                                                    "properties": {
                                                                                                                        "type": {
                                                                                                                            "type": "string"
                                                                                                                        }
                                                                                                                    },
                                                                                                                    "type": "object"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        },
                                                                                                        "required": {
                                                                                                            "items": {
                                                                                                                "type": "string"
                                                                                                            },
                                                                                                            "type": "array"
                                                                                                        },
                                                                                                        "type": {
                                                                                                            "type": "string"
                                                                                                        }
                                                                                                    },
                                                                                                    "type": "object"
                                                                                                },
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        }
                                                                                    },
                                                                                    "type": "object"
                                                                                },
                                                                                "required": {
                                                                                    "items": {
                                                                                        "type": "string"
                                                                                    },
                                                                                    "type": "array"
                                                                                },
                                                                                "type": {
                                                                                    "type": "string"
                                                                                }
                                                                            },
                                                                            "type": "object"
                                                                        },
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "Offset": {
                                                                    "properties": {
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "RecognitionStatus": {
                                                                    "properties": {
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "SpeakerId": {
                                                                    "properties": {},
                                                                    "type": "object"
                                                                }
                                                            },
                                                            "type": "object"
                                                        },
                                                        "required": {
                                                            "items": {
                                                                "type": "string"
                                                            },
                                                            "type": "array"
                                                        },
                                                        "type": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "type": "object"
                                                },
                                                "type": {
                                                    "type": "string"
                                                }
                                            },
                                            "type": "object"
                                        }
                                    },
                                    "type": "object"
                                },
                                "required": {
                                    "items": {
                                        "type": "string"
                                    },
                                    "type": "array"
                                },
                                "type": {
                                    "type": "string"
                                }
                            },
                            "type": "object"
                        },
                        "type": {
                            "type": "string"
                        }
                    },
                    "type": "object"
                }
            },
            "type": "object"
        },
        "type": {
            "type": "string"
        }
    },
    "type": "object"
}

```
You can rename this action for clarity.
Click on "**Add an action**"


![springle](Pictures/093.png)

In the search field, enter "**Compose**" and then click "**Compose**"


![springle](Pictures/094.png)

So we're going to prepare the JSON file and then send it to Azure Cosmos DB. With the "*Compose*" action, we will format the JSON file according to our needs. For example, I'm going to add a field that I'm going to name "MyDate" to create the partition key for recording results in the Azure Cosmos DB collection.

In "**Inputs**" field, paste the JSON script below:

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
  "MyDate":"",
  "Language": "English",
  "id": ""
}
```

![springle](Pictures/095.png)


Now we're going to complete the values for *Channel_0, Channel_1, Date, MyDate* and *id* Fields.

Click to the right of **"Channel_0":** and **just before** the comma, then add the field "**Body**" which is in "**Dynamic content**" under "**Parse channel 0 results**"

![springle](Pictures/096.png)

Do the same with **"Channel_1"** but this time taking the field "**Body**" under "**Parse channel 1 results**"

![springle](Pictures/097.png)


To the right of "**AudioLink**", between double quotes (""), add "**recordingsUrl**", which is in "**Dynamic content**" under "**Parse JSON 2**"

![springle](Pictures/098.png)

To the right of **"Date:"**, between double quotes (""), add **utcNow()** expression. Click on "**OK**".

![springle](Pictures/099.png)

To the right of  **"MyDate:"**, between double quotes (""), add  **formatDateTime(utcNow(),'yyyy-MM-dd')** expression. Click on "**OK**".

![springle](Pictures/100.png)

To the right of **"id:"**, between double quotes (""), add **body('Parse_JSON_2')[0]['id']** expression. Click "**OK**".

![springle](Pictures/101.png)

So you need to get something similar to the screen copy below. You can also rename this action for clarity. Here, I renamed this step "**Compose JSON document with 2 audio channels**". (We will use what we have just done at this stage for the "**if false**" branch for the second channel test)

Click "**Add an action**"

![springle](Pictures/102.png)

Add "**Parse JSON**" action, 

![springle](Pictures/103.png)


In "**Content**" field, add "**Outputs**" which is under"**Compose JSON document with 2 audio channels**"

![springle](Pictures/104.png)


In "**Schema**" field, add the scripts below



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
        "MyDate": {
            "type": "string"
        },
        "Language": {
            "type": "string"
        },
        "id": {
            "type": "string"
        }
    },
    "type": "object"
}
```

You should get a result similar to the screen copy below.

Click "**Add an action**"

![springle](Pictures/105.png)

Now that we've retrieved the information for both channels of our audio file, we'll record the results in an Azure Cosmos DB collection.

In the search field, enter the word "**Cosmos**" and then select "**Create or update document**"

![springle](Pictures/106.png)

Name your login and select the Azure Cosmos DB account you want to use. Click the button "**Create**"

![springle](Pictures/107.png)


In drop-down lists "**Database ID**" and "**Collection ID**", select information from your Azure Cosmos DB database and collection. Click on "**Document**" field then select "**Body**" under "**Parse JSON 3**"

![springle](Pictures/108.png)

Click "**Add new parameter**" field, then select "**Partition key value**"


![springle](Pictures/109.png)



Click on "**Partition key value**".

"**WARNING!!!!**" **Add double quotes** to surround the dynamic content of the "**MyDate**" that is under "**Parse JSON 3**" 


![sparkle](Pictures/110.png)

Nous allons maintenant traiter le cas des fichiers audio avec juste un seul cannal. Pour ce faire, nouys allons simplement renseigner la partie "*if false*" du test du second canal

Sous la partie "**If false**", cliquez sur "**Add an action**".

![sparkle](Pictures/111.png)

Dans le champ de recherche, entrez "**compose**", puis cliquez sur "**Compose**"

![sparkle](Pictures/112.png)

Collez le script ci-dessous dans le champ "**Inputs**"

```javascript
{
  "AudioDetails": [
    {
      "Channel_0":
      
    }
  ],
  "AudioLink": "",
  "Date": "",
  "MyDate":"",
  "Language": "English",
  "id": ""
}
```
Répétez les étapes vu précédement pour l'étape "**Compose JSON document with 2 audio channels**" mais pour un cannal seulement. Vous devez obtenir quelque chose de similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/113.png)

Cliquez sur "**Add an action**"

![sparkle](Pictures/114.png)

Puis dans le champ de recherche entrez "**parse**", puis sélectionnez "**Parse JSON**"

![sparkle](Pictures/115.png)

Cliquez dans le champ "**Content**" puis sur "**Dynamic content**". Séctionnez "**Ouputs**" sous "**Compose JSON document with just 1 audio channel**"

![sparkle](Pictures/116.png)

Cliquez dans le champ "**Schema**" et collez le script ci-dessous


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
        "MyDate": {
            "type": "string"
        },
        "Language": {
            "type": "string"
        },
        "id": {
            "type": "string"
        }
    },
    "type": "object"
}
```

Vous devriez obtenir quelque chose de similaire à la copie d'écran ci-dessous.
Cliquez sur "**Add an action**" 

![sparkle](Pictures/117.png)

Dans la zone de recherche, entrez "**cosmos**", puis cliquez sur "**Create or update document**" puis entrez les informations comme nous l'avons vu précédement dans cet article. 

![sparkle](Pictures/118.png)

**Attention** de prendre "**Body**" et "**Mydate**" de l'étape "**Parse JSON 4**". 
Vous devriez obtenir quelque chose de similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/119.png)

Nous allons maintenant rajouter la dernière étape de notre Logic App. Cette étape consiste à supprimer les informations de transcription qui restent stockés au niveau du service cognitif afin de laisser la place libre pour la prochaine transcription.

Cliquez sur le bouton "**New step**"

![sparkle](Pictures/120.png)

Dans la zone de recherche, entrez "**http**" puis cliquez sur "**HTTP**"

![sparkle](Pictures/121.png)

Dans le champ "**Method**", sélectionnez "**DELETE**".
Cliquez dans le champ URI, puis cliquez sur "**Expression**". Collez le script ci-dessous. Cliquez sur le bouton "**OK**"


```javascript
concat('https://<your-cognitive-service-region>.cris.ai/api/speechtotext/v2.0/Transcriptions/',body('Parse_JSON')[0]['id'])
```

![sparkle](Pictures/122.png)


Remplacez "<your-cognitive-service-region"> par la region de votre service cognitif. Dans mon cas, c'est *canadacentral*

Enfin, dans le champ "**header**", rajoutez les information suivantes :

| Content-Type              |                 application/json |
| :------------------------ | -------------------------------: |
| Ocp-Apim-Subscription-Key | *la clef de votre service cognitif*|

Comme illustré ci-dessous

![sparkle](Pictures/123.png)

Cliquez sur le bouton "**Save**"

![sparkle](Pictures/124.png)

# Test de la solution

Maintenant que nous avons terminé notre logic app, nous allons nous assurer que notre service est bien dans le status "**Enabled**" 


![sparkle](Pictures/129.png)



Dans le cas contraire, cliquez sur le bouton "**Enable**"

![sparkle](Pictures/130.png)

Avec Azure Storage Explorer, déposez le fichier audio, qui se trouve dans le dossier AudioFile, dans votre compte de stockage. Votre Azure Logic App doit alors se déclencher et commencer le process de transcription du fichier audio


![sparkle](Pictures/131.png)

Si tout va bien, l'opération doit se dérouler avec succes

![sparkle](Pictures/132.png)

Cliquez sur "**Succeeded**" (ou "Failed" suivant les cas :)) afin d'avoir plus de  détails 

![sparkle](Pictures/133.png)

Allez voir le résultat dans votre collection Azure Cosmos DB.

![sparkle](Pictures/134.png)


# Power BI
Maintenant que les données du fichier sont stockées dans une base Azure Cosmos DB, il est naturel de vouloir visualiser ces données via Power BI. Je ne vais pas détailler comment faire le rapport mais juste illustrer comment démarrer. Et je vous partage aussi un rapport que j'ai fait, pour exemple.

Depuis Power BI Desktop, cliquez sur "**Get Data**" puis sur "**More**"

![sparkle](Pictures/137.png)

Cliquez sur "**Azure**" puis sur "**Azure Cosmos DB**". Cliquez sur le bouton "**Connect**"

![sparkle](Pictures/138.png)

Renseignez l'URL de votre compte Azure Cosmos DB (que vous pouvez retrouver sur le portail Azure), puis cliquez sur le bouton "**OK**"

![sparkle](Pictures/139.png)

Entrez la clef de votre compte Azure Cosmos DB (que vous pouvez retrouver sur le portail Azure). Cliquez sur le bouton "**Connect**"

![sparkle](Pictures/140.png)

Vous devriez être capable de naviguer dans vos bases de données et collections. Choisissez votre collection et cliquez sur le bouton "**Transform Data**"

![sparkle](Pictures/141.png)

Dans Power Quesry Editor, cliquez sur les flèches pour accéder aux champs de votre collection

![sparkle](Pictures/142.png)

Choisissez les champs dont vous avez besoin et cliquez sur le bouton "**OK**"

![sparkle](Pictures/143.png)

Pour les colonnes qui ont encore les doubles flèches, cliquez dessus pour choisir les champs que vous souhaitez intégrer dans le rapport. Il est possible de répéter cette opération plusieurs fois, tout dépend de la profondeur de la structure du document.

![sparkle](Pictures/144.png)

Ci-dessous, un exemple de rapport Power BI (disponible sous le dossier [*Power BI*](https://github.com/franmer2/speechtotextfr/tree/master/Power%20BI)). J'ai rajouté la possibilité de faire un "**Drillthrough**" au niveau des vues générales des transcriptions afin d'avoir le détail des conversations.

![sparkle](Pictures/902.png)


Pour représenter la discussion dans le rapport, j'ai utilisé une visualisation de type Diagramme de Gantt

![sparkle](Pictures/145.png)


# Sous le capot

Vous devez peut-être vous poser la question à propos des longs scripts que l'on doit rajouter lors des étapes "Parse JSON" et comment les générer. Pour ce faire, j'ai utilisé l'outil [Postman](https://www.postman.com/) (mais je ne dis pas que c'est la meilleure façon de faire). Avec Postman, j'ai envoyé manuellement un fichier audio au service cognitif. Toujours avec l'outil Postman, je récupère les résultats au format JSON.

Ci-dessous la configuration de Postman pour envoyer un fichier audio au service cognitif.

La configuration de la partie "**Headers**"

![sparkle](Pictures/125.png)

Un exemple de configuration de la partie "**Body**"

![sparkle](Pictures/126.png)

Ensuite je récupére le résultat avec un "Get"

![sparkle](Pictures/127.png)

Que je copie dans la partie "**Use sample payload to generate schema**" de l'étape "**Parse JSON**" d'Azure Logic App

![sparkle](Pictures/128.png)

# Troubleshooting

Une des erreurs que j'ai eu lors des tests, était dû au fait que le service cognitif conservait les informations des fichiers audio précédent. C'est pour cette raison que la dernière étape du Logic App est une action "Delete" au niveau du service cognitif.

Dans le cas où vous avez une erreur avec Logic app, vous pouvez vérifier avec Postman, avec une commande "GET", si des informations sont encore présentes au niveau du service cognitif. Si c'est le cas, copiez l'id du document retourné.

![sparkle](Pictures/135.png)


Puis utilisez cet id pour envoyer une commande "DELETE" avec Postman, de la manière illustrée ci-dessous :


![sparkle](Pictures/136.png)



