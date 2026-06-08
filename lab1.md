# flaws.cloud — Level 1

## Vulnerability
S3 bucket with public LIST permissions — anyone can list bucket contents
without AWS credentials.

## What I did
1. Read the hint — learned flaws.cloud is hosted on S3 in us-west-2
2. Listed the bucket contents with no credentials:

    aws s3 ls s3://flaws.cloud --no-sign-request

## What I found
Secret file hidden in the bucket: secret-dd02c7c.html
Not linked anywhere on the site — visible because bucket allows public listing.

## The fix
Disable public LIST access on S3 buckets.
S3 → Permissions → Block Public Access → enable all four settings.

## Key takeaway
LIST permission alone exposes every filename in a bucket.
A bucket doesn't need to be fully public to leak data.

## Completed
Level 1 done. Secret file: secret-dd02c7c.html
Next: http://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud


# flaws.cloud — Level 2

## Vulnerability
S3 bucket open to any authenticated AWS user — not just the bucket owner.
"Everyone authenticated" is not the same as "private."

## What I did
1. Configured AWS CLI with a proper IAM user (not root — learned that lesson first)
2. Listed the bucket using my AWS credentials:

    aws s3 ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud

## What I found
Secret file: secret-e4443fc.html
Accessible to any AWS account in the world.

## The fix
Never grant s3:ListBucket to "Any authenticated AWS user."
That means 200+ million AWS accounts can access your bucket.
Treat it the same as public.

## Key takeaway
"Authenticated" does not mean "authorised."
Always scope bucket permissions to specific AWS account IDs or IAM roles.


# flaws.cloud — Level 3

## Vulnerability
Git repository accidentally exposed in a public S3 bucket.
AWS credentials committed to git history — never truly deleted.

## What I did
1. Listed the bucket and spotted a .git/ folder
2. Downloaded the entire bucket with git sync
3. Ran git log — found a commit called "Oops, accidentally added something I shouldn't have"
4. Ran git show on the previous commit and found access_keys.txt

## What I found
access_key REDACTED
secret_access_key REDACTED

## The fix
Never commit credentials to git — ever.
Use tools like git-secrets or truffleHog to scan repos for leaked keys.
If keys are leaked — rotate them immediately, git history cannot be cleaned easily.

## Key takeaway
Deleting a file from git does not delete it from history.
Always assume anything ever committed to a repo is permanent.