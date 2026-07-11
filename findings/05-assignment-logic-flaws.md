[← Back to overview](../README.md)

# Finding 5: Assignment Logic Flaws - Unenforced Scoring, Unlimited Retakes, and Weak Essay Grading

**Severity:** Low &nbsp;|&nbsp; **CVSS v3.1:** 4.3 (`AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:L/A:N`)
**CWE:** CWE-840 - Business Logic Errors
**OWASP Top 10 (2021):** A04:2021 Insecure Design
**Proof captured:** Yes (marker value redacted)

## Description

The assignment system has several related logic flaws. The submission endpoint
**POST /api/v1/assignments/1/submit** returns a protected marker in the raw response body
regardless of score; it never appears in the web UI and is only visible by inspecting the
response. Submitting deliberately wrong or low-scoring answers still returns it - a submission
scoring 32.6% with 3 of 9 MCQs correct returned it in full. Once an assignment is marked
complete, calling the start endpoint again creates a fresh submission, allowing unlimited
retakes. The essay grader also awards a high score to trivial input - a single-character essay
scored roughly 88% - and submissions are accepted with no minimum time.

## Reproduction Steps

1. Start the assignment: **POST /api/v1/assignments/1/start**

   ![Assignment dashboard before submission](../evidence/Finding-05-Assignment/SS-F05-01_assignment-dashboard.png)
   *__Figure 5.1__ - Assignment dashboard before submission.*

2. Submit with any answers, including all wrong ones: **POST /api/v1/assignments/1/submit**

   ![Submission confirmation in the web UI](../evidence/Finding-05-Assignment/SS-F05-05_assignment-submit-confirm.png)
   *__Figure 5.2__ - Submission confirmation in the web UI.*

3. The protected marker is returned in the raw JSON response regardless of score (visible in Burp).
4. Call start again on the same assignment to obtain a new **submission_id** and retake without limit.

   ![Start endpoint called again after completion](../evidence/Finding-05-Assignment/SS-F05-13_assignment-retake-unlimited.png)
   *__Figure 5.3__ - Start endpoint called again after completion.*

   ![New submission issued, confirming unlimited retakes](../evidence/Finding-05-Assignment/SS-F05-14_assignment-retake-confirmed.png)
   *__Figure 5.4__ - New submission issued, confirming unlimited retakes.*

## Business Impact

Anyone who reaches the submission endpoint obtains the marker, and unlimited
retakes plus weak essay grading make the leaderboard score worthless as a measure of anything. An
user who knows this can score high without doing the work, which is unfair to other candidates
and defeats the assessment. The likelihood is high, but the flaw only damages the fairness of the
assessment, not access to other users' data or the system itself, so it is rated Low.

## Remediation

Score submissions server-side and only return any completion token once a defined
pass threshold is met. Reject any start call once a submission exists for that user and assignment.
Replace the lenient essay scoring with a credible grading method and enforce a minimum engagement
time.

---

[← Finding 4](04-stored-xss.md) &nbsp;|&nbsp; [Back to overview](../README.md) &nbsp;|&nbsp; [Next: Finding 6 - JWT Refresh Token →](06-jwt-refresh-token.md)
