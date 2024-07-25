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



import os
from github import Github
from collections import Counter

# Get GitHub token from environment variables
token = os.getenv("GITHUB_TOKEN")
g = Github(token)

# Specify the repository owner
owner = "subhamNRchoudhary"

# Define project categories based on file extensions
categories = {
    'Python': ['.py'],
    'SQL': ['.sql'],
    'Power BI': ['.pbix'],
    'Tableau': ['.twb', '.twbx'],
    'Excel': ['.xlsx', '.xls']
}

# Function to determine the category of a repository based on its contents
def categorize_repo(repo):
    contents = repo.get_contents("")
    file_names = [content.path for content in contents if content.type == 'file']
    for file_name in file_names:
        for category, extensions in categories.items():
            if any(file_name.endswith(ext) for ext in extensions):
                return category
    return None

# Count projects in each category
project_counts = Counter()

# List repositories for the specified owner
repos = g.get_user(owner).get_repos()
for repo in repos:
    category = categorize_repo(repo)
    if category:
        project_counts[category] += 1

# Generate markdown content
markdown_content = "# Projects Overview\n\n"
markdown_content += "Here is an overview of the different types of projects in my GitHub repository:\n\n"

for category, count in project_counts.items():
    badge = f"https://img.shields.io/badge/{category.replace(' ', '%20')}-{count}-blue"
    markdown_content += f"![{category} Projects]({badge})\n\n"

# Update README.md
with open("README.md", "r") as readme:
    content = readme.readlines()

start_index = content.index("# Projects Overview\n")
end_index = content.index("<!-- PROJECT-STATS-END -->\n", start_index)

content = content[:start_index + 1] + [markdown_content] + content[end_index:]

with open("README.md", "w") as readme:
    readme.writelines(content)
