import re
import requests

# ---------------- CONFIG ----------------
JIRA_URL = "http://localhost:8080"
JIRA_USER = "admin"
JIRA_PASS = "admin"
PROJECT_KEY = "ABC"
RELEASE_VERSION = "v1.0.0"
# ----------------------------------------

# Commit messages (for demo, these are hardcoded)
commit_messages = [
    "ABC-1 Create login API",
    "ABC-2 Add validations"
]

auth = (JIRA_USER, JIRA_PASS)
headers = {"Content-Type": "application/json"}

# Step 1: Extract Jira keys
jira_issues = set()
for msg in commit_messages:
    matches = re.findall(rf"{PROJECT_KEY}-\d+", msg)
    jira_issues.update(matches)

print("Issues found:", jira_issues)

# Step 2: Create Version in Jira
version_payload = {
    "name": RELEASE_VERSION,
    "project": PROJECT_KEY
}

r = requests.post(
    f"{JIRA_URL}/rest/api/2/version",
    auth=auth,
    headers=headers,
    json=version_payload
)
print("Created version:", r.status_code)

# Step 3: Update issues with Fix Version
for issue in jira_issues:
    payload = {
        "update": {
            "fixVersions": [{"add": {"name": RELEASE_VERSION}}]
        }
    }

    r = requests.put(
        f"{JIRA_URL}/rest/api/2/issue/{issue}",
        auth=auth,
        headers=headers,
        json=payload
    )

    print(f"Updated {issue}: {r.status_code}")

print("DONE")