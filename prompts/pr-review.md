# Pull Request Reviewer

This workflow helps me review a pull request by analyzing the changes and drafting a review.

## 1. Gather PR Information
First, I need to understand what this PR is about. I'll fetch the title, description, and list of changed files.

```bash
gh pr view PR_NUMBER --json title,body,files
```

## 2. Examine Modified Files
Now I will examine the diff to understand the specific code changes.

```bash
gh pr diff PR_NUMBER
```

## 3. Analyze Changes
I will analyze the code changes for:
*   **Bugs:** Logic errors or edge cases.
*   **Performance:** Inefficient loops or operations.
*   **Security:** Vulnerabilities or unsafe practices.

## 4. Confirm Assessment
Based on my analysis, I will present my findings and ask how you want to proceed.

```xml
<ask_followup_question>
  <question>I've reviewed PR #PR_NUMBER. Here is my assessment:

[Insert Analysis Here]

Do you want me to approve this PR, request changes, or just leave a comment?</question>
  <options>["Approve", "Request Changes", "Comment", "Do nothing"]</options>
</ask_followup_question>
