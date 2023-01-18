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
    - GH_TOKEN  with a value that corresponds to a [GitHub Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).Make sure the token scopes with the minimum required permissions (repo)



## Running the webservice:
- Start the local web service via `flask run --host=0.0.0.0 --port=<port>` in case `--port` isn't used the default port is 5000
- Start forwarding the service via `./ngrok http <port>` in case `--port` in the command above wasn't used run: `./ngrok http 5000 `. Take note of the url returned by ngrok for the next step.
- Create new [Webhook](https://docs.github.com/en/developers/webhooks-and-events/webhooks/about-webhooks) on the organisation level as follow:
    - Payload URL: the url returned by ngrok
    - Content type: change to: application/json
    - Secret: while it's a good practice to use a secret for security reasons to make sure github only is calling the webservice - this is currently not implemented, so for now leave this empty
    - SSL verification: Leave enabled
    - Events that should trigger the Webhook - select the following individual events:
        - Issues
        - Repositories
        - Repositories imports
    - Check the Active
    - Note that every time ngrok is restarted a new url will be created and updating the webhook url is required