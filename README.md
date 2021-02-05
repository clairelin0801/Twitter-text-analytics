# Twitter-text-analytics
{
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "actions": {
            "偵測情感": {
                "inputs": {
                    "body": {
                        "text": "@triggerBody()?['TweetText']"
                    },
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['cognitiveservicestextanalytics']['connectionId']"
                        }
                    },
                    "method": "post",
                    "path": "/sentiment"
                },
                "runAfter": {},
                "type": "ApiConnection"
            },
            "條件": {
                "actions": {
                    "傳送簡訊_(SMS)": {
                        "inputs": {
                            "body": {
                                "body": "The score is @{body('偵測情感')?['id']}. The tweet was @{triggerBody()?['TweetText']}.",
                                "from": "+16506847097",
                                "to": "+8860975292824"
                            },
                            "host": {
                                "connection": {
                                    "name": "@parameters('$connections')['twilio']['connectionId']"
                                }
                            },
                            "method": "post",
                            "path": "/Messages.json"
                        },
                        "runAfter": {},
                        "type": "ApiConnection"
                    }
                },
                "expression": {
                    "and": [
                        {
                            "less": [
                                "@body('偵測情感')?['score']",
                                "@float(0.5)"
                            ]
                        }
                    ]
                },
                "runAfter": {
                    "偵測情感": [
                        "Succeeded"
                    ]
                },
                "type": "If"
            }
        },
        "contentVersion": "1.0.0.0",
        "outputs": {},
        "parameters": {
            "$connections": {
                "defaultValue": {},
                "type": "Object"
            }
        },
        "triggers": {
            "有新推文張貼時": {
                "inputs": {
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['twitter']['connectionId']"
                        }
                    },
                    "method": "get",
                    "path": "/onnewtweet",
                    "queries": {
                        "searchQuery": "matching"
                    }
                },
                "recurrence": {
                    "frequency": "Minute",
                    "interval": 5
                },
                "splitOn": "@triggerBody()?['value']",
                "type": "ApiConnection"
            }
        }
    },
    "parameters": {
        "$connections": {
            "value": {
                "cognitiveservicestextanalytics": {
                    "connectionId": "/subscriptions/ceb878f1-87ab-478f-b122-09f72dca4dbf/resourceGroups/first/providers/Microsoft.Web/connections/cognitiveservicestextanalytics",
                    "connectionName": "cognitiveservicestextanalytics",
                    "id": "/subscriptions/ceb878f1-87ab-478f-b122-09f72dca4dbf/providers/Microsoft.Web/locations/eastasia/managedApis/cognitiveservicestextanalytics"
                },
                "twilio": {
                    "connectionId": "/subscriptions/ceb878f1-87ab-478f-b122-09f72dca4dbf/resourceGroups/first/providers/Microsoft.Web/connections/twilio",
                    "connectionName": "twilio",
                    "id": "/subscriptions/ceb878f1-87ab-478f-b122-09f72dca4dbf/providers/Microsoft.Web/locations/eastasia/managedApis/twilio"
                },
                "twitter": {
                    "connectionId": "/subscriptions/ceb878f1-87ab-478f-b122-09f72dca4dbf/resourceGroups/first/providers/Microsoft.Web/connections/twitter",
                    "connectionName": "twitter",
                    "id": "/subscriptions/ceb878f1-87ab-478f-b122-09f72dca4dbf/providers/Microsoft.Web/locations/eastasia/managedApis/twitter"
                }
            }
        }
    }
}
