# How to detect if a build is a restart or a new build

The CLI ```codefresh get build``` does not provide the information but the API does:

```
get_status:
  image: <image>
  commands:
    - |-
      export BUILD_STRATEGY=$(curl -X GET -H "Authorization: ${{CF_API_KEY}}" "${{CF_URL}}/api/builds/${{CF_BUILD_ID}}" | jq -r '.buildStrategy')
    - echo $BUILD_STRATEGY  
```
It will return "fresh-start", "restart-failed-steps" or "restart"
