# Connect-the-other

name: Update Project Stats

on:
  schedule:
    - cron: '0 0 * * *'  # Runs daily at midnight
  workflow_dispatch:

jobs:
  update-readme:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install PyGithub

      - name: Update README with project stats
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: python update_readme.py






with open("README.md", "w") as readme:
    readme.writelines(content)
