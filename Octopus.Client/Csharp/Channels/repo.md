Sure, we can enhance the script to include information about teams and contributors for each repository. Here's how you can do it:

### Step 1: Update the Script to Fetch Team and Contributor Data

We will make additional API calls to fetch teams and contributors for each repository. This will require more API requests, so be mindful of the rate limits.

### Updated Script

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
repositories = fetch_paginated_data(org_repos_url)

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
        
        # Fetch teams for the repository
        repo_teams_url = f'https://api.github.com/repos/{ORG_NAME}/{repo_name}/teams'
        teams = fetch_paginated_data(repo_teams_url)
        team_names = [team['name'] for team in teams]

        # Fetch contributors for the repository
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

1. **Pagination**: The function `fetch_paginated_data` handles paginated responses from the GitHub API.
2. **Teams and Contributors**: For each repository, additional API calls fetch the teams and contributors associated with the repository.
3. **Rate Limiting**: A small delay (`time.sleep(1)`) between requests helps to avoid hitting the rate limit.

### Running the Script

1. Save the updated script to a file, e.g., `generate_github_report_with_teams_and_contributors.py`.
2. Replace `'your_personal_access_token'` and `'your_organization_name'` with your actual GitHub token and organization name.
3. Run the script:

```sh
python generate_github_report_with_teams_and_contributors.py
```

This will generate a CSV file named `github_repositories_with_teams_and_contributors.csv` with details of all repositories, including their associated teams and contributors.

Let me know if you need any further adjustments or additional details in the report!
