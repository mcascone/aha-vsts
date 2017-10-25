# aha-vsts
API commands used to integrate VSTS and Aha! in ways the native integration doesn't do.

## GET get aha release fields
Get the metadata for the release you want to put the feature into. Use any AHA_NUM from that release.

```java
{{AHA_URL}}/releases/{{AHA_REL}}/
```

## GET get tfs item info
```java
{{VSTS_URL}}/workitems/{{VSTS_ID}}?api-version=1.0&$expand=none```
