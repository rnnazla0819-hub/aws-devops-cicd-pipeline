5# AWS DevOps & CI/CD: Module 06

**Intern:** Raheem Noorul Nazla
**Program:** AWS Cloud & DevOps Internship, Codomax Digital Solutions
**Module:** 06, AWS DevOps & CI/CD

## Project overview

A basic CI/CD pipeline. Every push to the `main` branch starts a GitHub Actions workflow that checks the website files and syncs them to an Amazon S3 bucket with static website hosting.

```
Developer -> git push (main) -> GitHub Actions -> check files -> aws s3 sync -> S3 static website
```

**Live website:** `http://nazla-cicd-demo-2026.s3-website-ap-southeast-2.amazonaws.com/`

## Objectives covered

1. Git and GitHub fundamentals
2. Version control (commits, branches, pull requests, history)
3. Continuous Integration and Continuous Deployment
4. AWS CI/CD concepts (CodePipeline, CodeBuild, CodeDeploy)
5. A basic deployment pipeline
6. Documented deployment workflow (this file)

## Repository structure

```
.
├── .github/workflows/deploy.yml   # the pipeline
├── site/index.html                # the website that gets deployed
└── README.md                      # this documentation
```

## Pipeline steps (`deploy.yml`)

| Step | What it does |
|------|--------------|
| Trigger | Runs on every push to `main` (or by hand from the Actions tab) |
| Check out the code | Downloads the repository into the runner |
| Check the site files | Fails the run if `site/index.html` is missing, so nothing broken is deployed |
| Configure AWS credentials | Reads the AWS keys from GitHub Secrets |
| Sync to S3 | `aws s3 sync site/ s3://<bucket> --delete` uploads changes and removes deleted files |

## One-time setup

### 1. S3 bucket
- Create a new bucket (example name: `nazla-cicd-demo-2026`) in `ap-southeast-2`.
- Turn off "Block all public access" for this website bucket only.
- Properties > Static website hosting > Enable, index document `index.html`.
- Add a public read bucket policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicReadGetObject",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::nazla-cicd-demo-2026/*"
  }]
}
```

### 2. IAM user for deployment (least privilege)
Create a user such as `github-deployer` with only this inline policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::nazla-cicd-demo-2026"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::nazla-cicd-demo-2026/*"
    }
  ]
}
```
Create an access key for this user.

### 3. GitHub Secrets
Repository > Settings > Secrets and variables > Actions > New repository secret:

| Secret | Value |
|--------|-------|
| `AWS_ACCESS_KEY_ID` | the access key ID |
| `AWS_SECRET_ACCESS_KEY` | the secret access key |

## How to deploy

1. Create a branch, edit `site/index.html`, and commit.
2. Open a pull request and merge it into `main`.
3. The push to `main` starts the workflow. Open the **Actions** tab to watch the run.
4. Open the S3 website endpoint to see the change.

## How to roll back

Use `git revert <commit>` (or the Revert button on a merged pull request) and merge it. The pipeline redeploys the earlier version.

## Security notes

- Keys are stored only in GitHub Secrets, never in the code or screenshots.
- The IAM user can access one bucket only.
- After the review, delete the access key in IAM (Security credentials) and delete the bucket if it is no longer needed.

## Screenshots

(Add your screenshots to a `screenshots/` folder and link them here.)

- Repository and commit history
- Pull request merged
- GitHub Actions run (success)
- S3 bucket with the deployed files
- Live website

## Key learnings

- A CI/CD pipeline makes every release the same automatic, repeatable process.
- Every deployment is a commit, so the history shows who changed what and when.
- Secrets and least-privilege IAM permissions keep an automated pipeline safe.
