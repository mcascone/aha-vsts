# API commands to integrate VSTS and Aha! in ways the native integration doesn't do.

## Workflow:
1. Get the Aha release you want to add the new feature to.
2. Create new feature in AHA.
3. Create the VSTS integration link in it, pointing to the VSTS item.
    * This can be run on an existing Aha item to create the integration link to VSTS.
5. Update the VSTS item with the AHA-ID and a link to it.
6. Add a blank description to the VSTS item if necessary - Aha requires this field to exist for the integration to work.


----

### get aha release fields
Get the metadata for the release you want to put the feature into. Use any AHA_NUM from that release.
```javascript
GET {{AHA_URL}}/releases/{{AHA_REL}}/
```

### GET get tfs item info
```javascript
GET {{VSTS_URL}}/workitems/{{VSTS_ID}}?api-version=1.0&$expand=none
```

### get aha integration id
```javascript
GET {{AHA_URL}}/products/{{AHA_PROD_ID}}/integrations
```

### POST Create Aha Feature w/TFS info
Creates a new feature in the Aha release specified in the previous step. Captures the new feature num in {{AHA_NUM}}
```javascript
POST {{AHA_URL}}/releases/{{AHA_REL_ID}}/features
```
BODY
```json 
{
	"feature": 
	{
		"name": "{{TFS_NAME}}",
		"type": "Enhancement",
		"workflow_status": {
			"name": "{{AHA_STATE}}"
		}
	},
	"description": "{{TFS_DESC}}"
}
```


### add tfs link to aha
```java
POST {{AHA_URL}}/{{AHA_TYPE}}s/{{AHA_NUM}}/integrations/{{AHA_INT_ID}}/fields
```
BODY
```json
{
	"integration_fields": [
	{
		"name": "id",
		"value": "{{VSTS_ID}}"
	},
	{
		"name": "url",
		"value": "{{VSTS_INT_URL}}={{VSTS_ID}}&_a=edit"
	}
	]
}
```

 
### add link and aha field to tfs
This seems to break the state linkage if it's in a state that doesn't line up well between the two. Mostly based on the split columns, they don't map well. Double check that it doesn't slip when using this one.
```json
PATCH {{VSTS_URL}}/DefaultCollection/_apis/wit/workitems/{{VSTS_ID}}?api-version=1.0
```
BODY
```json
[
 {
	"op": "add",
	"path": "/fields/Consilio_Agile.AhaID",
	"value": "{{AHA_NUM}}"
  },
  {
	"op": "add",
	"path": "/relations/-",
	"value": {
		"rel": "hyperlink",
		"url": "https://consilio.aha.io/{{AHA_TYPE}}s/{{AHA_NUM}}"
	}
  }
]
```

### add blank description to tfs
In cases where there's no Decription info in TFS for Aha to pull in, the call fails. This is a quick way to add a blank character to the TFS item so the other API calls go through.
```json
PATCH {{VSTS_URL}}/DefaultCollection/_apis/wit/workitems/{{VSTS_ID}}?api-version=1.0
```
BODY
```json
[
 {
	"op": "add",
	"path": "/fields/System.Description",
	"value": "<br>"
  }
]
```
