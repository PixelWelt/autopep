# AutoPEP8 GitHub Action

This repository contains a GitHub Action workflow that automatically checks Python code for PEP8 compliance on every pull request to the `main` branch.

## Workflow Overview

The workflow defined in `autopep.yml` performs the following steps:

1. **Set up Python**: Uses the `actions/setup-python` action to set up a Python environment.
2. **Install pycodestyle**: Installs the `pycodestyle` package, which is used to check Python code against PEP8 style conventions.
3. **Run pycodestyle**: Runs `pycodestyle` on the Python files that have changed in the pull request and saves the output to a file.
4. **Read pycodestyle output**: Reads the output from the `pycodestyle` run and sets it as an output variable.
5. **Create comment**: Uses the `actions/github-script` action to create a comment on the pull request with the PEP8 issues found.

## Usage

To use this workflow in your repository, follow these steps:

1. Copy the `autopep.yml` file to the `.github/workflows` directory in your repository.
2. Ensure you have a `GITHUB_TOKEN` secret set up in your repository settings.

## Example `autopep.yml`

```yaml
on: 
    pull_request:
        branches:
            - main

jobs:
    generate_comment:
        runs-on: ubuntu-latest
        name: An example job to comment a PR
        steps:
            - name: Set up Python
                uses: actions/setup-python@v2
                with:
                    python-version: '3.x'
            - name: Install pycodestyle
                run: pip install pycodestyle
            - name: Run pycodestyle
                id: pycodestyle
                run: |
                 git diff --name-only ${{ github.event.before }} ${{ github.sha }} | grep '\.py$' | xargs pycodestyle --statistics > pep8_issues.txt || true
            - name: Read pycodestyle output
                id: read_pep8_issues
                run: echo "::set-output name=pep8_issues::$(cat pep8_issues.txt)"
            - name: Create comment
                uses: actions/github-script@v3
                with:
                    github-token: ${{ secrets.GITHUB_TOKEN }}
                    script: |
                            const pep8_issues = `\${{ steps.read_pep8_issues.outputs.pep8_issues }}`;
                            github.rest.issues.createComment({
                                issue_number: context.issue.number,
                                owner: context.repo.owner,
                                repo: context.repo.repo,
                                body: 'Hey there after checking your code, I\'ve found the following pep8 issues. Please fix them. <br> ```' + pep8_issues + '```'
                            })
```

## Contributing

Contributions are welcome! Please open an issue or submit a pull request if you have any improvements or suggestions.
