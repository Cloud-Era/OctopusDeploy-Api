To see the script's progress and what it is doing while running, we can add print statements that log the current status of the script to the terminal. This will give you real-time feedback on which repository is being processed and when the script is fetching teams and contributors.

Here's the updated script with logging:

```python
import requests
import csv
import time

# Replace these with your own values
GITHUB_TOKEN = 'your_personal_access_token'
ORG_NAME = 'your_organization_name'

headers = {
    'Authorization': f'token {GITHUB_TOKEN}'
}

# GitHub API endpoints
org_repos_url = f'https://api.github.com/orgs/{ORG_NAME}/repos'
teams_url = f'https://api.github.com/orgs/{ORG_NAME}/teams'

# Function to fetch paginated data from GitHub API
def fetch_paginated_data(url, params={}):
    data = []
    page = 1
    while True:
        response = requests.get(url, headers=headers, params={**params, 'per_page': 100, 'page': page})
        page_data = response.json()
        if not page_data:
            break
        data.extend(page_data)
        page += 1
        time.sleep(1)  # To avoid hitting the rate limit
    return data

# Fetch all repositories in the organization
print("Fetching all repositories...")
repositories = fetch_paginated_data(org_repos_url)
print(f"Total repositories found: {len(repositories)}")

# Define the CSV file path
csv_file_path = 'github_repositories_with_teams_and_contributors.csv'

# Define the CSV headers
csv_headers = ['Repository Name', 'Description', 'Private', 'URL', 'Created At', 'Updated At', 'Pushed At', 'Size', 'Stargazers Count', 'Watchers Count', 'Forks Count', 'Teams', 'Contributors']

# Write data to CSV
with open(csv_file_path, mode='w', newline='', encoding='utf-8') as csv_file:
    writer = csv.DictWriter(csv_file, fieldnames=csv_headers)
    writer.writeheader()
    for repo in repositories:
        repo_name = repo['name']
        print(f"Processing repository: {repo_name}")

        # Fetch teams for the repository
        print(f"  Fetching teams for repository: {repo_name}")
        repo_teams_url = f'https://api.github.com/repos/{ORG_NAME}/{repo_name}/teams'
        teams = fetch_paginated_data(repo_teams_url)
        team_names = [team['name'] for team in teams]

        # Fetch contributors for the repository
        print(f"  Fetching contributors for repository: {repo_name}")
        repo_contributors_url = f'https://api.github.com/repos/{ORG_NAME}/{repo_name}/contributors'
        contributors = fetch_paginated_data(repo_contributors_url)
        contributor_logins = [contributor['login'] for contributor in contributors]

        writer.writerow({
            'Repository Name': repo['name'],
            'Description': repo['description'],
            'Private': repo['private'],
            'URL': repo['html_url'],
            'Created At': repo['created_at'],
            'Updated At': repo['updated_at'],
            'Pushed At': repo['pushed_at'],
            'Size': repo['size'],
            'Stargazers Count': repo['stargazers_count'],
            'Watchers Count': repo['watchers_count'],
            'Forks Count': repo['forks_count'],
            'Teams': ', '.join(team_names),
            'Contributors': ', '.join(contributor_logins)
        })

print(f"Report generated: {csv_file_path}")
```

### Explanation

1. **Fetching Repositories**: A print statement announces the fetching of all repositories.
2. **Repository Processing**: For each repository, print statements indicate the repository being processed and when teams and contributors are being fetched.
3. **Final Report**: A final print statement announces the generation of the report.

### Running the Script

1. Save the updated script to a file, e.g., `generate_github_report_with_logging.py`.
2. Replace `'your_personal_access_token'` and `'your_organization_name'` with your actual GitHub token and organization name.
3. Run the script:

```sh
python generate_github_report_with_logging.py
```

This will provide real-time feedback on the terminal, showing the script's progress as it processes each repository, fetches teams, and fetches contributors.
