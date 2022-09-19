# Source code scanning


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

