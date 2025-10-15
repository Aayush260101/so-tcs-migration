🧩 Our Use Case: Manual Release Gate in GitHub Actions
We built a GitHub Actions workflow where:

Jobs job1 to job4 run first.

job5 waits for a manual trigger before proceeding.

The goal was to simulate a release gate that could be manually activated after upstream jobs complete.

⚠️ Key Limitation
GitHub Actions does not support pausing a workflow mid-execution and resuming it later from the same point. Once a workflow starts, it must run continuously to completion. This limitation meant we couldn’t insert a true “wait-for-user-trigger” step inside a running workflow.

🔄 Our Chosen Alternative: File-Based Trigger
To work around this, we implemented a file-based trigger:

job5 enters a polling loop and checks for the existence of a file (release_trigger.txt) in the repository.

Once the file is detected via GitHub’s API, the job proceeds.

🛠️ What We Did
1️⃣ Triggered the Workflow Manually
We used the workflow_dispatch API to start the workflow:

cmd
curl --ssl-no-revoke -X POST ^
  -H "Authorization: token YOUR_GITHUB_TOKEN" ^
  -H "Accept: application/vnd.github+json" ^
  https://api.github.com/repos/OWNER/REPO/actions/workflows/WORKFLOW_FILE_NAME/dispatches ^
  -d "{\"ref\":\"main\"}"
2️⃣ Simulated Failure in job4
We added random failure logic to job4 so we could test retrying failed jobs.

3️⃣ Retried Failed Jobs via API
When job4 failed, we retried it using GitHub’s rerun-failed-jobs API:

cmd
curl --ssl-no-revoke -X POST ^
  -H "Authorization: token YOUR_GITHUB_TOKEN" ^
  -H "Accept: application/vnd.github+json" ^
  https://api.github.com/repos/OWNER/REPO/actions/runs/RUN_ID/rerun-failed-jobs
4️⃣ Unblocked job5 by Uploading a Trigger File
Once all jobs succeeded, we manually created release_trigger.txt using GitHub’s content API:

cmd:-
curl --ssl-no-revoke -X PUT ^
  -H "Authorization: token YOUR_GITHUB_TOKEN" ^
  -H "Accept: application/vnd.github+json" ^
  -d "{\"message\":\"Release Job 5\",\"content\":\"BASE64_ENCODED_CONTENT\"}" ^
  https://api.github.com/repos/OWNER/REPO/contents/release_trigger.txt
✅ Final Outcome :-
We successfully triggered the workflow manually.
Simulated failure in job4 and retried it via API.
Used a file-based trigger to manually release job5.
The workflow completed as expected after detecting the trigger file.
