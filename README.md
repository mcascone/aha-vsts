# API commands used to integrate VSTS and Aha! in ways the native integration doesn't do.

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


### GET get tfs item info

### GET get aha integration id

### POST Create Aha Feature w/TFS info
Creates a new feature in the Aha release specified in the previous step. Captures the new feature num in {{AHA_NUM}}

### POST add tfs link to aha
 
### PATCH add link and aha field to tfs
This seems to break the state linkage if it's in a state that doesn't line up well between the two. Mostly based on the split columns, they don't map well. Double check that it doesn't slip when using this one.

### PATCH add blank description to tfs
In cases where there's no Decription info in TFS for Aha to pull in, the call fails. This is a quick way to add a blank character to the TFS item so the other API calls go through.
