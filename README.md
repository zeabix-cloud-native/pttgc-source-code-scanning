# Source code scanning

## PREREQUISITE
### Create Repository Secrets
`REGISTRY_LOGIN_SERVER`
`REGISTRY_USERNAME`
`REGISTRY_PASSWORD`

1. Login to Azure portal 

[Azure_Portal](https://azure.microsoft.com/en-us/get-started/azure-portal/)

2. Select Azure cloud Shell
<img width="1103" alt="Screen Shot 2565-08-23 at 20 35 17" src="https://user-images.githubusercontent.com/46469458/186177823-b282391b-3737-4e11-90ea-e1137378f0ef.png">

3. First command to login to ACR. 
 
 ```console
 
export RESOURCE_GROUP="$(az group list --query "[?location=='eastasia']" | jq -r '.[0].name')"

 groupId=$(az group show \
   --name ${RESOURCE_GROUP} \
   --query id --output tsv)
 ```

4. Second command to login to ACR. create the service principal (Copy clientId and clientSecret please see detail in step 7.)
 
  ```console
 az ad sp create-for-rbac \
  --scope $groupId \
  --role Contributor \
  --sdk-auth
  ```
 
 <img width="761" alt="Screen Shot 2565-08-23 at 22 35 05" src="https://user-images.githubusercontent.com/46469458/186200948-9cfecd01-e02e-4fa1-a861-d2a8fb24e64c.png">

5. Third command to login to ACR. Please change <registry-name> to your registry name from step 4.
 
 ```console
 export REPO_NAME="$(az acr list | jq -r '.[].name')"
 ```

```console
 registryId=$(az acr show \
   --name ${REPO_NAME} \
   --query id --output tsv)
 ```
   
 
6. Fourth command to login to ACR. Please change <ClientId> to your clientId or app id from step 4. And please keep the result.

  ```console
 az role assignment create \
  --assignee <ClientId> \
  --scope $registryId \
  --role AcrPush
  ```
 
<img width="1512" alt="Screen Shot 2565-08-23 at 20 04 07" src="https://user-images.githubusercontent.com/46469458/186165619-ac871267-2a51-4aed-bc55-60612e7e48c7.png">
 
7. Get `REGISTRY_LOGIN_SERVER`
 
   ```console
   az acr list | jq -r '.[].loginServer'
   ```
 
8. Create Github Repo.

 In the GitHub UI, navigate to your forked repository and select Settings > Secrets > Actions and Select New repository secret to add the following secrets:

![image](https://user-images.githubusercontent.com/46469458/190967242-e4224148-6a45-4588-8415-f8451e8d431d.png)

Reference : https://docs.microsoft.com/en-us/azure/container-instances/container-instances-github-action

---

## Exercise 1
### Set up basic CodeQL
1. In the github repository to go menu `Security-->Code scanning`
2. Click `Configure CodeQL alerts`
3. (Optional) It will populate basic workflow with codeQL, adjust it as needed and then start commit


## Exercise 2
### Integrate with and existing workflow
1. In `.github/workflows/build-workflow.yml` add the CodeQL job between `unittest` and `containerized`

```
sourcecode-scanning:
    name: Scan source code with CodeQL 
    runs-on: ubuntu-latest
    strategy:
      matrix: 
        language: ['csharp']
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      - name: Initial CodeQL
        uses: github/codeql-action/init@v2
        with:
          languages: ${{ matrix.language }}

      - name: Autobuild
        uses: github/codeql-action/autobuild@v2

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2
        with:
          category: "/language:${{matrix.language}}"
```

2. Commit and Push the code and see how the workflow is running

