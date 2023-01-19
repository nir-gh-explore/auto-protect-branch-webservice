# Auto-protect-branch-webservice

Auto branch protect is a simple web service that listens for organization events to know when a repository has been created. 
When the repository is created this web service automates the protection of the main branch. It also notifies you with an @mention in an issue within the repository that outlines the protections that were added.

## Initial Environment Setup:
- Install the following:
    - [Python](https://www.python.org/downloads/)
        - `pip install -r requirements.txt`
    - [Flask](https://flask.palletsprojects.com/en/2.2.x/installation/)
    - [ngrok](https://ngrok.com/download)

- Set the following environment variables:
    - GH_USER with the value of the username which the service would be using to call the API's for branch protection and the issue
    - GH_TOKEN  with a value that corresponds to a [GitHub Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).
Make sure the token scopes with the minimum required permissions (repo)



## Running the webservice:
- Start the local web service via `flask run --host=0.0.0.0 --port=<port>` in case `--port` isn't used the default port is `5000`
- Start forwarding the service via `./ngrok http <port>` in case `--port` in the command above wasn't used run: `./ngrok http 5000 `.
- Copy the `Forwarding` url returned by ngrok for the next step. For example if the following is retuned by ngrok command:
`Forwarding                    https://b5ca-175-39-127-114.au.ngrok.io -> http://localhost:5678` - copy the `https://b5ca-175-39-127-114.au.ngrok.io`
- Create new [Webhook](https://docs.github.com/en/developers/webhooks-and-events/webhooks/about-webhooks) on the organisation level as follow:
    - Payload URL: the url returned by ngrok
    - Content type: change to: application/json
    - Secret: while it's a good practice to use a secret for security reasons to make sure github only is calling the webservice - this is currently not implemented, so for now leave this empty
    - SSL verification: Leave enabled
    - Events that should trigger the Webhook - select the following individual events:
        - Issues
        - Repositories
        - Repositories imports
    - Make sure the Webhook is set to 'Active' by ticking the checkbox and save.
- Note that every time ngrok is restarted a new url will be created and updating the webhook url is required
- Create a new repository:
    - Make sure the repo is public to enable protection for a GitHub free plan
    - Add at least a Readme file so that the repo isn't empty and there is main branch
- Verify that the [branch protection](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/managing-a-branch-protection-rule), and a new [issue](https://docs.github.com/en/issues/tracking-your-work-with-issues/about-issues) were created as needed.
- Troubleshooting and debugging could be done by setting up the following variable before running the webservice: `FLASK_DEBUG=1`

## Resources and attributions:
- The code in this repo is based on the [following archived repo](https://github.com/zkoppert/Auto-branch-protect/blob/main/README.md) with some improvements:
    - Some of the dependencies in the Setup required update as there were conflicts
    - Dependabot and CodeQL scanning were enabled and the security issues that were detected are now remidiated
    - Performance and documentation improvements
    - Replaced the `.gitignore` file with a more robust file generated automatically by GitHub
- A couple of great resources to get familiar with webhooks and REST are
    - The [following workshop](https://github.com/githubsatelliteworkshops/webhooks-with-rest)
    - There is a live webinar which walks through [here](https://www.youtube.com/watch?v=wcxOJq9YemE)
- [GithHub REST API documentation](https://docs.github.com/en/rest?apiVersion=2022-11-28)
    - [Branch Protection](https://docs.github.com/en/rest/branches/branch-protection?apiVersion=2022-11-28)
    - [Create an Issue](https://docs.github.com/en/rest/issues/issues?apiVersion=2022-11-28#create-an-issue)
    - [Delete Repository](https://docs.github.com/en/rest/issues/issues?apiVersion=2022-11-28#create-an-issue) - used for cleaning up the test repos
- There is the following [postman collection](https://github.com/nir-gh-explore/GH-PostmanCollection) that was created to explore the 3 APIs above

## Future Improvements:
- Implement a secret for the webhook and support it via the webservice
- As demonstrated in [this commit](https://github.com/nir-gh-explore/auto-protect-branch-webservice/commit/c422b305c6c7764a0b0b6a650acddfe91ca8bcf5) the way that the branch protection url is constructed following this commit fixes the SSRF security issue for the first API. The same pattern could be applied to fix the issue creation API call as it's currently vulnarable to SSRF
- An alternative solution may be implementing similar functionality using a [templated GitHub Action](https://github.com/orgs/community/discussions/25748)
