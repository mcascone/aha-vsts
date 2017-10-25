# API commands to integrate VSTS and Aha! in ways the native integration doesn't do.

## Workflow:
1. Get the Aha release you want to add the new feature to.
2. Create new feature in AHA.
3. Create the VSTS integration link in it, pointing to the VSTS item.
    * This can be run on an existing Aha item to create the integration link to VSTS.
5. Update the VSTS item with the AHA-ID and a link to it.
6. Add a blank description to the VSTS item if necessary - Aha requires this field to exist for the integration to work.


----
---

### GET get aha release fields
Get the metadata for the release you want to put the feature into. Use any AHA_NUM from that release.
```javascript
GET {{AHA_URL}}/releases/{{AHA_REL}}/
```
Postman test code to capture return data:
```javascript
var jsonData = JSON.parse(responseBody);

postman.setGlobalVariable("AHA_REL_ID", jsonData.release.id);

// needed to get the integration
postman.setGlobalVariable("AHA_PROD_ID", jsonData.release.product_id);
```

---
### GET get tfs item info
```javascript
GET {{VSTS_URL}}/workitems/{{VSTS_ID}}?api-version=1.0&$expand=none
```
Postman test code to capture return data:
```javascript
var jsonData = JSON.parse(responseBody);

postman.setGlobalVariable("TFS_NAME", jsonData.fields["System.Title"]);

// Get the assignee and strip out the email address
var assigned = jsonData.fields["System.AssignedTo"].split(" <")[0];
postman.setGlobalVariable("TFS_ASSIGNED_TO", assigned);


// if this comes out as undefined, need to use the set function to set a blank desc.
// Otherwise, updating TFS from AHA will not work (TFS will fail the request from AHA).
// Would be better to catch this and send a description automatically.
// postman.setGlobalVariable("TFS_DESC", JSON.stringify(jsonData.fields["System.Description"]));
// postman.setGlobalVariable("TFS_DESC_PARSED", JSON.parse(postman.getGlobalVariable("TFS_DESC")));
postman.setGlobalVariable("TFS_DESC", jsonData.fields["System.Description"]);

var tfsState = jsonData.fields["System.State"];
var ahaState;

var tfsType = jsonData.fields["System.WorkItemType"];
var ahaFeatType;

switch (tfsState) {
    case "Ready": 
        ahaState = "Backlog";
        break;

    case "Active":
        ahaState = "In Development";
        break;
    
    case "Resolved":
        ahaState = "Ready To Ship";
        break;
        
    case "In UAT":
        ahaState = "In UAT";
        break;
        
    case "Closed":
        ahaState = "Shipped";
        break;
        
    default:
        ahaState = "Backlog";
}

if ( (jsonData.fields["System.BoardColumn"] == "Active") && (jsonData.fields["System.BoardColumnDone"] === true)) {
        ahaState = "In QA";
}


switch (tfsType) {
    case "Bug":
        ahaFeatType = "Bug";
        // the body of the item is in a different place for TFS Bugs
        postman.setGlobalVariable("TFS_DESC", jsonData.fields["Microsoft.VSTS.TCM.ReproSteps"]);
        break;
    
    default:
        ahaFeatType = "Enhancement";
}

postman.setGlobalVariable("AHA_STATE", ahaState);
postman.setGlobalVariable("AHA_FEAT_TYPE", ahaFeatType);
```


---
### GET get aha integration id
```javascript
GET {{AHA_URL}}/products/{{AHA_PROD_ID}}/integrations
```
Postman Test code to capture return data
```javascript
var jsonData = JSON.parse(responseBody);

// there might be multiple integrations, so only find the one for TFS
for (var i in jsonData.integrations) {
    if (jsonData.integrations[i].service_name === "tfs") {
        postman.setGlobalVariable("AHA_INT_ID", jsonData.integrations[i].id);
    }
}
```
---
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
Postman Test code - captures return data
```javascript
var jsonData = JSON.parse(responseBody);

// capture the new feature's reference number
postman.setGlobalVariable("AHA_NUM", jsonData.feature.reference_num);
postman.setGlobalVariable("AHA_ITEM_URL", jsonData.feature.url);
```


---
### POST add tfs link to aha
```javascript
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

---
### PATCH add link and aha field to tfs
This seems to break the state linkage if it's in a state that doesn't line up well between the two. Mostly based on the split columns, they don't map well. Double check that it doesn't slip when using this one.
```javascript
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
---
### PATCH add blank description to tfs
In cases where there's no Decription info in TFS for Aha to pull in, the call fails. This is a quick way to add a blank character to the TFS item so the other API calls go through.
```javascript
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
