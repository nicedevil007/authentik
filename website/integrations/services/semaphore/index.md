---
title: Integrate with Semaphore UI
sidebar_label: semaphore
---

# Semaphore UI

<span class="badge badge--secondary">Support level: Community</span>

## What is Semaphore UI

> Semaphore UI is a modern web interface for managing popular DevOps tools.
> -- https://semaphoreui.com/
>
> This guide explains how to set up Semaphore UI to use authentik as the OAuth provider for the login to the WebGUI.

## Preparation

The following placeholders will be used:

- `semaphore.company` is the FQDN of the engomo install.
- `authentik.company` is the FQDN of the authentik install.
- `ak.cert` is the self-signed certificate that will be used for the service provider.

## Semaphore UI configuration

Create an application and an OAuth2/OpenID provider in authentik. Use the following parameters for the OAuth2/OpenID provider:

**Provider:**

- Name: `SP-semaphore`
- Client type: `Confidential`
- Redirect URIs/Origins (RegEx): `https://semaphore.company/api/auth/oidc/authentik/redirect/`
- Signing Key: `ak.cert`
- Scopes: `authentik default OAuth Mapping: OpenID 'email', OpenID 'openid'` and `OpenID 'profile'`

Take note of the Client ID and Client Secret, you'll need to give them to Semaphore UI in Step 3.

Leave the rest as default values. The durations can be changed as needed.

**Application:**

- Name: `Semaphore UI`
- Slug: `semaphore`
- Launch URL: `https://semaphore.company/`

## Semaphore UI configuration

Login to your Semaphore UI host via SSH. Edit the `config.json` (should be located under `/etc/semaphore`) file with the texteditor of your choice.
For example just use this command:

```bash
nano /etc/semaphore/config.json
```

Before the last curly brace, add this part.

```
"oidc_providers": {
        "authentik": {
                "display_name": "SSO-Login",
                "provider_url": "https://authentik.company/application/o/semaphore/",
                "client_id": "<<< Client ID >>>",
                "client_secret": "<<< Client Secret >>>",
                "redirect_url": "https://semaphore.company/api/auth/oidc/authentik/redirect/",
                "username_claim": "username",
                "name_claim": "name",
                "email_claim": "email",
                "scopes": ["openid", "profile", "email"]
        }
}
```

:::note It is mandatory to include 'authentik' in lowercase letters. There should also be another curly brace above these lines. Make sure to add a `,` after it to maintain proper formatting. :::

More information on this can be found in the semaphore documentation https://docs.semaphoreui.com/administration-guide/openid/authentik/.

Leave the rest as default.

## Test the login

- Open a browser of your choice and open the URL `https://semaphore.company`.
- Click on the SSO-Login button.
- You should be redirected to authentik (with the login flows you created) and then authentik should redirect you back to `https://semaphore.company` URL.
- If you are redirected back to the `https://semaphore.company` URL you did everything correct.

:::note Users are created upon logging in with authentik. They will not have the rights to create anything initially. These permissions must be assigned later by the local admin created during the first login to the Semaphore UI. :::
