# Introduction

> How to set up Open SWE for development

# Development Setup Overview

Welcome to the Open SWE development setup guide. This section will walk you through everything you need to know to get Open SWE running locally for development.

## Setup Sections

The setup process is organized into focused sections to help you get up and running efficiently:

<CardGroup cols={2}>
  <Card title="Development Setup" icon="code" href="/labs/swe/setup/development">
    Complete guide to cloning the repository, installing dependencies,
    configuring environment variables, and starting the development servers.
  </Card>

  <Card title="Authentication" icon="shield-check" href="/labs/swe/setup/authentication">
    Understanding the authentication flow, GitHub App configuration, and
    security mechanisms used throughout Open SWE.
  </Card>
</CardGroup>

## Technology Stack

Open SWE is built with modern technologies designed for scalability and developer experience:

### Core Technologies

* **TypeScript** - Strict type safety across the entire codebase
* **Yarn** - Package manager with workspace support (v3.5.1)
* **Turbo** - Monorepo build orchestration and task running

### Agent Infrastructure

* **LangGraph** - Multi-agent orchestration framework with three specialized graphs:
  * Manager graph for user interaction orchestration
  * Planner graph for execution plan creation
  * Programmer graph for code change execution
* **Daytona** - Sandboxed development environments for safe code execution

### Web Application

* **Next.js** - React framework with App Router
* **Shadcn UI** - Component library built on Radix UI primitives
* **Tailwind CSS** - Utility-first CSS framework

### Documentation

* **Mintlify** - Documentation platform with MDX support

## Monorepo Structure

<Note>
  Open SWE uses a Yarn workspace monorepo with three main applications and a
  shared package for common utilities and types.
</Note>

* **`apps/open-swe`** - LangGraph agent application
* **`apps/web`** - Next.js web interface
* **`apps/docs`** - Mintlify documentation site
* **`packages/shared`** - Shared utilities, types, and constants

The monorepo is orchestrated by Turbo, which handles build dependencies and parallel task execution across packages.


# Development Setup

> How to set up Open SWE for development

This guide will walk you through setting up Open SWE for local development. You'll need to clone the repository, install dependencies, configure environment variables, create a GitHub App, and start the development servers.

<Note>
  This setup is for development purposes. For production deployment, you'll need
  to adjust URLs and create separate GitHub Apps for production use.
</Note>

## Prerequisites

Before starting, ensure you have the following installed:

* Node.js (version 18 or higher)
* Yarn (version 3.5.1 or higher)
* Git

## Setup Steps

<Steps>
  <Step title="Clone the Repository">
    Clone the Open SWE repository to your local machine:

    ```bash
    git clone https://github.com/langchain-ai/open-swe.git
    ```

    ```bash
    cd open-swe
    ```
  </Step>

  <Step title="Install Dependencies">
    Install all dependencies using Yarn from the repository root:

    ```bash
    yarn install
    ```

    This will install dependencies for all packages in the monorepo workspace.
  </Step>

  <Step title="Set Up Environment Files">
    Copy the environment example files and configure them:

    ```bash
    # Copy web app environment file
    cp apps/web/.env.example apps/web/.env
    ```

    ```bash
    # Copy agent environment file
    cp apps/open-swe/.env.example apps/open-swe/.env
    ```

    ### Web App Environment Variables (`apps/web/.env`)

    Fill in the following variables (GitHub App values will be added in the next step):

    ```bash Environment Variables [expandable]
    # API URLs for development
    NEXT_PUBLIC_API_URL="http://localhost:3000/api"
    LANGGRAPH_API_URL="http://localhost:2024"

    # Encryption key for secrets (generate with: openssl rand -hex 32)
    SECRETS_ENCRYPTION_KEY=""

    # GitHub App OAuth settings (will be filled after creating GitHub App)
    NEXT_PUBLIC_GITHUB_APP_CLIENT_ID=""
    GITHUB_APP_CLIENT_SECRET=""
    GITHUB_APP_REDIRECT_URI="http://localhost:3000/api/auth/github/callback"

    # GitHub App details (will be filled after creating GitHub App)
    GITHUB_APP_NAME="open-swe-dev" # this must match the name of your GitHub app, excluding spaces
    GITHUB_APP_ID=""
    GITHUB_APP_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
    ...add your private key here...
    -----END RSA PRIVATE KEY-----
    "

    # List of GitHub usernames that are allowed to use Open SWE without providing API keys
    # This is only used in production. In development every user is an "allowed user".
    # Must be a valid JSON array of strings.
    NEXT_PUBLIC_ALLOWED_USERS_LIST='["your-github-username", "teammate-username"]'
    ```

    ### Agent Environment Variables (`apps/open-swe/.env`)

    Configure the agent environment variables:

    ```bash Environment Variables [expandable]
    # LangSmith tracing & LangGraph platform
    LANGCHAIN_PROJECT="default"
    LANGCHAIN_API_KEY="lsv2_pt_..."  # Get from LangSmith
    LANGCHAIN_TRACING_V2="true"
    LANGCHAIN_TEST_TRACKING="false"

    # LLM Provider Keys (at least one required)
    ANTHROPIC_API_KEY=""  # Recommended - default provider
    OPENAI_API_KEY=""     # Optional
    GOOGLE_API_KEY=""     # Optional

    # Infrastructure
    DAYTONA_API_KEY=""    # Required. For cloud sandboxes

    # Tools
    FIRECRAWL_API_KEY=""  # For URL content extraction

    # GitHub App settings (same as web app)
    GITHUB_APP_NAME="open-swe-dev" # this must match the name of your GitHub app, excluding spaces
    GITHUB_APP_ID=""
    GITHUB_APP_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
    ...add your private key here...
    -----END RSA PRIVATE KEY-----
    "
    GITHUB_WEBHOOK_SECRET=""  # Will be generated in next step

    # Server configuration
    PORT="2024"
    OPEN_SWE_APP_URL="http://localhost:3000"
    SECRETS_ENCRYPTION_KEY=""  # Must match web app value

    # CI/CD. See the /labs/swe/setup/ci for more information
    SKIP_CI_UNTIL_LAST_COMMIT="true"

    # List of GitHub usernames that are allowed to use Open SWE without providing API keys
    # This is only used in production. In development every user is an "allowed user".
    # Must be a valid JSON array of strings.
    NEXT_PUBLIC_ALLOWED_USERS_LIST='["your-github-username", "teammate-username"]'
    ```

    If you don't set the `NEXT_PUBLIC_ALLOWED_USERS_LIST` environment variable, every user will be required to set their own LLM API keys to use the agent. Additionally, none of the webhook features (triggering new runs by adding labels to GitHub issues, tagging the agent in PR reviews, etc.) will work unless you include the username's of the GitHub users you want to allow access to in the `NEXT_PUBLIC_ALLOWED_USERS_LIST` environment variable (in both web and agent deployments).

    <Tip>
      Generate the `SECRETS_ENCRYPTION_KEY` using: `openssl rand -hex 32`. This key must be identical in both environment files.
    </Tip>
  </Step>

  <Step title="Create GitHub App">
    <Note>
      You'll need to create a **GitHub App** (not a GitHub OAuth App). These are different types of applications with different capabilities. Consider creating separate GitHub apps for development and production environments.
    </Note>

    ### Create the GitHub App

    1. Go to [GitHub App creation page](https://github.com/settings/apps/new)
    2. Fill in the basic information:
       * **GitHub App name**: Your preferred name
       * **Description**: Development instance of Open SWE coding agent
       * **Homepage URL**: Your repository URL
       * **Callback URL**: `http://localhost:3000/api/auth/github/callback`

    ### Configure OAuth Settings

    * ✅ **Request user authorization (OAuth) during installation** - Allows users to log in to the web app
    * ✅ **Redirect on update** - Redirects users back to your app after permission updates
    * ❌ **Expire user authorization tokens** - Keep tokens from expiring

    ### Set Up Webhook

    1. ✅ **Enable webhook**

    2. **Webhook URL**: You'll need to use a tool like ngrok to expose your local server:
       ```bash
       # Install ngrok if you haven't already
       # Then expose your local LangGraph server
       ngrok http 2024
       ```
       Use the ngrok URL + `/webhooks/github` (e.g., `https://abc123.ngrok.io/webhooks/github`)

    3. **Webhook secret**: Generate and save this value:
       ```bash
       openssl rand -hex 32
       ```
       Add this value to `GITHUB_WEBHOOK_SECRET` in `apps/open-swe/.env`

    ### Configure Permissions

    **Repository permissions:**

    * **Contents**: Read & Write
    * **Issues**: Read & Write
    * **Pull requests**: Read & Write
    * **Metadata**: Read only (automatically enabled)

    **Organization permissions:** None

    **Account permissions:** None

    This gif shows how/where to enable the permissions on the GitHub App:

    <iframe className="w-full aspect-video rounded-xl" src="https://www.youtube.com/embed/rw6ddYmiTMo" title="Permissions Walkthrough" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowFullScreen />

    ### Subscribe to Events

    * ✅ **Issues** - Required for webhook functionality
    * ✅ **Pull request review** - Required for PR tagging functionality
    * ✅ **Pull request review comment** - Required for PR tagging functionality
    * ✅ **Issue comment** - Required for PR tagging functionality

    ### Installation Settings

    * **Where can this GitHub App be installed?**:
      * Choose "Any account" for broader testing
      * Or "Only on this account" to limit to your repositories

    ### Complete App Creation

    Click **Create GitHub App** to finish the setup.

    ### Collect App Credentials

    After creating the app, collect the following values and add them to both environment files:

    * **GITHUB\_APP\_NAME**: The name you chose
    * **GITHUB\_APP\_ID**: Found in the "About" section (e.g., `12345678`)
    * **NEXT\_PUBLIC\_GITHUB\_APP\_CLIENT\_ID**: Found in the "About" section
    * **GITHUB\_APP\_CLIENT\_SECRET**:
      1. Scroll to "Client secrets" section
      2. Click "Generate new client secret"
      3. Copy the generated value
    * **GITHUB\_APP\_PRIVATE\_KEY**:
      1. Scroll to "Private keys" section
      2. Click "Generate a private key"
      3. Download the `.pem` file and copy its contents
      4. Format as a single line with `\\n` for line breaks, or use the multiline format shown in the example
    * **GITHUB\_APP\_REDIRECT\_URI**: Should be `http://localhost:3000/api/auth/github/callback` for local development, or `https://your-production-url.com/api/auth/github/callback` for production.
    * **GITHUB\_WEBHOOK\_SECRET**: Generate and save this value:
      ```bash
      openssl rand -hex 32
      ```
      Add this value to `GITHUB_WEBHOOK_SECRET` in `apps/open-swe/.env` and `apps/web/.env`.

    Below are screenshots showing where in the GitHub App settings you can find the values for the environment variables.

    <Accordion title="GitHub App OAuth Settings">
      The following screenshots show where to find the following values:

      * `GITHUB_APP_ID`
      * `NEXT_PUBLIC_GITHUB_APP_CLIENT_ID`
      * `GITHUB_APP_CLIENT_SECRET`
      * `GITHUB_APP_REDIRECT_URI`
      * `GITHUB_APP_PRIVATE_KEY`

            <img src="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=fb991d989991d2caeb48ce3fbbc0a7c3" alt="GitHub Secrets Screenshot 1" width="1566" height="1270" data-path="images/gh_app_token_screenshot_1.png" srcset="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=280&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=cc5970b49d2e7ef10dd4f407d427cfef 280w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=560&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=4c45b7f451ffb0a97e92be99d440a85b 560w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=840&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=c54350d2e9ae420e7861506e85b5b883 840w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=1100&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=38ed6ead11f92edf356475720b518e03 1100w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=1650&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=0efde9106cba89a024fd78c935e73b73 1650w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=2500&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=e954ce21030c75c322a4068912c17539 2500w" data-optimize="true" data-opv="2" />
            <img src="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=c0e0c6e28a22433563c51a54170700b8" alt="GitHub Secrets Screenshot 2" width="1672" height="752" data-path="images/gh_app_token_screenshot_2.png" srcset="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=280&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=85b381cd5b4fbb3838640cf9f3c0de14 280w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=560&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=2b0ac74a8d588cc61221f3df9d11b310 560w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=840&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=139590cb21b2481512b4019993324bfc 840w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=1100&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=c6b259a0e88325fd1c56aa3748029927 1100w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=1650&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=f9e723a49554cb2f50d3b8551e2e1a74 1650w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=2500&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=e5f7675304d17b1a2426de55dbc02ab7 2500w" data-optimize="true" data-opv="2" />
            <img src="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=79ccc241a04c3ccdec140a7b2faf3746" alt="GitHub Secrets Screenshot 3" width="1652" height="656" data-path="images/gh_app_token_screenshot_3.png" srcset="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=280&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=04879f70660d41b91d1dac6ab799a863 280w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=560&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=6e75946858b954672ed6c65a8cf00d53 560w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=840&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=60f190da606dd9d0e1ef75f48d8c9ac3 840w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=1100&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=d7bf5b836e82880a8108f372c9bd101b 1100w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=1650&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=6229006d569d5c10063fb9b8e61f1c6b 1650w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=2500&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=8b0ff610c986a2201bd7d0f70711e979 2500w" data-optimize="true" data-opv="2" />
    </Accordion>

    <Tip>
      Keep your GitHub App credentials secure and never commit them to version control. The `.env` files are already included in `.gitignore`.
    </Tip>
  </Step>

  <Step title="Start Development Servers">
    With all environment variables configured, start both development servers:

    **Terminal 1 - Start the LangGraph Agent:**

    ```bash
    # apps/open-swe
    yarn dev
    ```

    This starts the LangGraph server at `http://localhost:2024`

    **Terminal 2 - Start the Web Application:**

    ```bash
    # apps/web
    yarn dev
    ```

    This starts the Next.js web app at `http://localhost:3000`

    <Note>
      Both servers need to be running simultaneously for full functionality. The web app communicates with the LangGraph agent through API calls.
    </Note>
  </Step>
</Steps>

## Verification

Once both servers are running:

1. **Visit the web app**: Navigate to `http://localhost:3000`
2. **Test GitHub authentication**: Try logging in with your GitHub account

<Tip>
  If you encounter issues, check the console logs in both terminal windows for
  error messages. Common issues include missing environment variables or
  incorrect GitHub App configuration.
</Tip>

## Next Steps

* Learn about [Authentication](/labs/swe/setup/authentication) to understand how the GitHub App integration works
* Explore [Usage](/labs/swe/usage/intro) to start using Open SWE for code changes
* Review the [Monorepo Structure](/labs/swe/setup/monorepo) for development best practices


# Development Setup

> How to set up Open SWE for development

This guide will walk you through setting up Open SWE for local development. You'll need to clone the repository, install dependencies, configure environment variables, create a GitHub App, and start the development servers.

<Note>
  This setup is for development purposes. For production deployment, you'll need
  to adjust URLs and create separate GitHub Apps for production use.
</Note>

## Prerequisites

Before starting, ensure you have the following installed:

* Node.js (version 18 or higher)
* Yarn (version 3.5.1 or higher)
* Git

## Setup Steps

<Steps>
  <Step title="Clone the Repository">
    Clone the Open SWE repository to your local machine:

    ```bash
    git clone https://github.com/langchain-ai/open-swe.git
    ```

    ```bash
    cd open-swe
    ```
  </Step>

  <Step title="Install Dependencies">
    Install all dependencies using Yarn from the repository root:

    ```bash
    yarn install
    ```

    This will install dependencies for all packages in the monorepo workspace.
  </Step>

  <Step title="Set Up Environment Files">
    Copy the environment example files and configure them:

    ```bash
    # Copy web app environment file
    cp apps/web/.env.example apps/web/.env
    ```

    ```bash
    # Copy agent environment file
    cp apps/open-swe/.env.example apps/open-swe/.env
    ```

    ### Web App Environment Variables (`apps/web/.env`)

    Fill in the following variables (GitHub App values will be added in the next step):

    ```bash Environment Variables [expandable]
    # API URLs for development
    NEXT_PUBLIC_API_URL="http://localhost:3000/api"
    LANGGRAPH_API_URL="http://localhost:2024"

    # Encryption key for secrets (generate with: openssl rand -hex 32)
    SECRETS_ENCRYPTION_KEY=""

    # GitHub App OAuth settings (will be filled after creating GitHub App)
    NEXT_PUBLIC_GITHUB_APP_CLIENT_ID=""
    GITHUB_APP_CLIENT_SECRET=""
    GITHUB_APP_REDIRECT_URI="http://localhost:3000/api/auth/github/callback"

    # GitHub App details (will be filled after creating GitHub App)
    GITHUB_APP_NAME="open-swe-dev" # this must match the name of your GitHub app, excluding spaces
    GITHUB_APP_ID=""
    GITHUB_APP_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
    ...add your private key here...
    -----END RSA PRIVATE KEY-----
    "

    # List of GitHub usernames that are allowed to use Open SWE without providing API keys
    # This is only used in production. In development every user is an "allowed user".
    # Must be a valid JSON array of strings.
    NEXT_PUBLIC_ALLOWED_USERS_LIST='["your-github-username", "teammate-username"]'
    ```

    ### Agent Environment Variables (`apps/open-swe/.env`)

    Configure the agent environment variables:

    ```bash Environment Variables [expandable]
    # LangSmith tracing & LangGraph platform
    LANGCHAIN_PROJECT="default"
    LANGCHAIN_API_KEY="lsv2_pt_..."  # Get from LangSmith
    LANGCHAIN_TRACING_V2="true"
    LANGCHAIN_TEST_TRACKING="false"

    # LLM Provider Keys (at least one required)
    ANTHROPIC_API_KEY=""  # Recommended - default provider
    OPENAI_API_KEY=""     # Optional
    GOOGLE_API_KEY=""     # Optional

    # Infrastructure
    DAYTONA_API_KEY=""    # Required. For cloud sandboxes

    # Tools
    FIRECRAWL_API_KEY=""  # For URL content extraction

    # GitHub App settings (same as web app)
    GITHUB_APP_NAME="open-swe-dev" # this must match the name of your GitHub app, excluding spaces
    GITHUB_APP_ID=""
    GITHUB_APP_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
    ...add your private key here...
    -----END RSA PRIVATE KEY-----
    "
    GITHUB_WEBHOOK_SECRET=""  # Will be generated in next step

    # Server configuration
    PORT="2024"
    OPEN_SWE_APP_URL="http://localhost:3000"
    SECRETS_ENCRYPTION_KEY=""  # Must match web app value

    # CI/CD. See the /labs/swe/setup/ci for more information
    SKIP_CI_UNTIL_LAST_COMMIT="true"

    # List of GitHub usernames that are allowed to use Open SWE without providing API keys
    # This is only used in production. In development every user is an "allowed user".
    # Must be a valid JSON array of strings.
    NEXT_PUBLIC_ALLOWED_USERS_LIST='["your-github-username", "teammate-username"]'
    ```

    If you don't set the `NEXT_PUBLIC_ALLOWED_USERS_LIST` environment variable, every user will be required to set their own LLM API keys to use the agent. Additionally, none of the webhook features (triggering new runs by adding labels to GitHub issues, tagging the agent in PR reviews, etc.) will work unless you include the username's of the GitHub users you want to allow access to in the `NEXT_PUBLIC_ALLOWED_USERS_LIST` environment variable (in both web and agent deployments).

    <Tip>
      Generate the `SECRETS_ENCRYPTION_KEY` using: `openssl rand -hex 32`. This key must be identical in both environment files.
    </Tip>
  </Step>

  <Step title="Create GitHub App">
    <Note>
      You'll need to create a **GitHub App** (not a GitHub OAuth App). These are different types of applications with different capabilities. Consider creating separate GitHub apps for development and production environments.
    </Note>

    ### Create the GitHub App

    1. Go to [GitHub App creation page](https://github.com/settings/apps/new)
    2. Fill in the basic information:
       * **GitHub App name**: Your preferred name
       * **Description**: Development instance of Open SWE coding agent
       * **Homepage URL**: Your repository URL
       * **Callback URL**: `http://localhost:3000/api/auth/github/callback`

    ### Configure OAuth Settings

    * ✅ **Request user authorization (OAuth) during installation** - Allows users to log in to the web app
    * ✅ **Redirect on update** - Redirects users back to your app after permission updates
    * ❌ **Expire user authorization tokens** - Keep tokens from expiring

    ### Set Up Webhook

    1. ✅ **Enable webhook**

    2. **Webhook URL**: You'll need to use a tool like ngrok to expose your local server:
       ```bash
       # Install ngrok if you haven't already
       # Then expose your local LangGraph server
       ngrok http 2024
       ```
       Use the ngrok URL + `/webhooks/github` (e.g., `https://abc123.ngrok.io/webhooks/github`)

    3. **Webhook secret**: Generate and save this value:
       ```bash
       openssl rand -hex 32
       ```
       Add this value to `GITHUB_WEBHOOK_SECRET` in `apps/open-swe/.env`

    ### Configure Permissions

    **Repository permissions:**

    * **Contents**: Read & Write
    * **Issues**: Read & Write
    * **Pull requests**: Read & Write
    * **Metadata**: Read only (automatically enabled)

    **Organization permissions:** None

    **Account permissions:** None

    This gif shows how/where to enable the permissions on the GitHub App:

    <iframe className="w-full aspect-video rounded-xl" src="https://www.youtube.com/embed/rw6ddYmiTMo" title="Permissions Walkthrough" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowFullScreen />

    ### Subscribe to Events

    * ✅ **Issues** - Required for webhook functionality
    * ✅ **Pull request review** - Required for PR tagging functionality
    * ✅ **Pull request review comment** - Required for PR tagging functionality
    * ✅ **Issue comment** - Required for PR tagging functionality

    ### Installation Settings

    * **Where can this GitHub App be installed?**:
      * Choose "Any account" for broader testing
      * Or "Only on this account" to limit to your repositories

    ### Complete App Creation

    Click **Create GitHub App** to finish the setup.

    ### Collect App Credentials

    After creating the app, collect the following values and add them to both environment files:

    * **GITHUB\_APP\_NAME**: The name you chose
    * **GITHUB\_APP\_ID**: Found in the "About" section (e.g., `12345678`)
    * **NEXT\_PUBLIC\_GITHUB\_APP\_CLIENT\_ID**: Found in the "About" section
    * **GITHUB\_APP\_CLIENT\_SECRET**:
      1. Scroll to "Client secrets" section
      2. Click "Generate new client secret"
      3. Copy the generated value
    * **GITHUB\_APP\_PRIVATE\_KEY**:
      1. Scroll to "Private keys" section
      2. Click "Generate a private key"
      3. Download the `.pem` file and copy its contents
      4. Format as a single line with `\\n` for line breaks, or use the multiline format shown in the example
    * **GITHUB\_APP\_REDIRECT\_URI**: Should be `http://localhost:3000/api/auth/github/callback` for local development, or `https://your-production-url.com/api/auth/github/callback` for production.
    * **GITHUB\_WEBHOOK\_SECRET**: Generate and save this value:
      ```bash
      openssl rand -hex 32
      ```
      Add this value to `GITHUB_WEBHOOK_SECRET` in `apps/open-swe/.env` and `apps/web/.env`.

    Below are screenshots showing where in the GitHub App settings you can find the values for the environment variables.

    <Accordion title="GitHub App OAuth Settings">
      The following screenshots show where to find the following values:

      * `GITHUB_APP_ID`
      * `NEXT_PUBLIC_GITHUB_APP_CLIENT_ID`
      * `GITHUB_APP_CLIENT_SECRET`
      * `GITHUB_APP_REDIRECT_URI`
      * `GITHUB_APP_PRIVATE_KEY`

            <img src="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=fb991d989991d2caeb48ce3fbbc0a7c3" alt="GitHub Secrets Screenshot 1" width="1566" height="1270" data-path="images/gh_app_token_screenshot_1.png" srcset="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=280&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=cc5970b49d2e7ef10dd4f407d427cfef 280w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=560&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=4c45b7f451ffb0a97e92be99d440a85b 560w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=840&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=c54350d2e9ae420e7861506e85b5b883 840w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=1100&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=38ed6ead11f92edf356475720b518e03 1100w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=1650&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=0efde9106cba89a024fd78c935e73b73 1650w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=2500&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=e954ce21030c75c322a4068912c17539 2500w" data-optimize="true" data-opv="2" />
            <img src="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=c0e0c6e28a22433563c51a54170700b8" alt="GitHub Secrets Screenshot 2" width="1672" height="752" data-path="images/gh_app_token_screenshot_2.png" srcset="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=280&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=85b381cd5b4fbb3838640cf9f3c0de14 280w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=560&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=2b0ac74a8d588cc61221f3df9d11b310 560w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=840&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=139590cb21b2481512b4019993324bfc 840w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=1100&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=c6b259a0e88325fd1c56aa3748029927 1100w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=1650&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=f9e723a49554cb2f50d3b8551e2e1a74 1650w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=2500&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=e5f7675304d17b1a2426de55dbc02ab7 2500w" data-optimize="true" data-opv="2" />
            <img src="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=79ccc241a04c3ccdec140a7b2faf3746" alt="GitHub Secrets Screenshot 3" width="1652" height="656" data-path="images/gh_app_token_screenshot_3.png" srcset="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=280&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=04879f70660d41b91d1dac6ab799a863 280w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=560&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=6e75946858b954672ed6c65a8cf00d53 560w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=840&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=60f190da606dd9d0e1ef75f48d8c9ac3 840w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=1100&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=d7bf5b836e82880a8108f372c9bd101b 1100w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=1650&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=6229006d569d5c10063fb9b8e61f1c6b 1650w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=2500&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=8b0ff610c986a2201bd7d0f70711e979 2500w" data-optimize="true" data-opv="2" />
    </Accordion>

    <Tip>
      Keep your GitHub App credentials secure and never commit them to version control. The `.env` files are already included in `.gitignore`.
    </Tip>
  </Step>

  <Step title="Start Development Servers">
    With all environment variables configured, start both development servers:

    **Terminal 1 - Start the LangGraph Agent:**

    ```bash
    # apps/open-swe
    yarn dev
    ```

    This starts the LangGraph server at `http://localhost:2024`

    **Terminal 2 - Start the Web Application:**

    ```bash
    # apps/web
    yarn dev
    ```

    This starts the Next.js web app at `http://localhost:3000`

    <Note>
      Both servers need to be running simultaneously for full functionality. The web app communicates with the LangGraph agent through API calls.
    </Note>
  </Step>
</Steps>

## Verification

Once both servers are running:

1. **Visit the web app**: Navigate to `http://localhost:3000`
2. **Test GitHub authentication**: Try logging in with your GitHub account

<Tip>
  If you encounter issues, check the console logs in both terminal windows for
  error messages. Common issues include missing environment variables or
  incorrect GitHub App configuration.
</Tip>

## Next Steps

* Learn about [Authentication](/labs/swe/setup/authentication) to understand how the GitHub App integration works
* Explore [Usage](/labs/swe/usage/intro) to start using Open SWE for code changes
* Review the [Monorepo Structure](/labs/swe/setup/monorepo) for development best practices


# Development Setup

> How to set up Open SWE for development

This guide will walk you through setting up Open SWE for local development. You'll need to clone the repository, install dependencies, configure environment variables, create a GitHub App, and start the development servers.

<Note>
  This setup is for development purposes. For production deployment, you'll need
  to adjust URLs and create separate GitHub Apps for production use.
</Note>

## Prerequisites

Before starting, ensure you have the following installed:

* Node.js (version 18 or higher)
* Yarn (version 3.5.1 or higher)
* Git

## Setup Steps

<Steps>
  <Step title="Clone the Repository">
    Clone the Open SWE repository to your local machine:

    ```bash
    git clone https://github.com/langchain-ai/open-swe.git
    ```

    ```bash
    cd open-swe
    ```
  </Step>

  <Step title="Install Dependencies">
    Install all dependencies using Yarn from the repository root:

    ```bash
    yarn install
    ```

    This will install dependencies for all packages in the monorepo workspace.
  </Step>

  <Step title="Set Up Environment Files">
    Copy the environment example files and configure them:

    ```bash
    # Copy web app environment file
    cp apps/web/.env.example apps/web/.env
    ```

    ```bash
    # Copy agent environment file
    cp apps/open-swe/.env.example apps/open-swe/.env
    ```

    ### Web App Environment Variables (`apps/web/.env`)

    Fill in the following variables (GitHub App values will be added in the next step):

    ```bash Environment Variables [expandable]
    # API URLs for development
    NEXT_PUBLIC_API_URL="http://localhost:3000/api"
    LANGGRAPH_API_URL="http://localhost:2024"

    # Encryption key for secrets (generate with: openssl rand -hex 32)
    SECRETS_ENCRYPTION_KEY=""

    # GitHub App OAuth settings (will be filled after creating GitHub App)
    NEXT_PUBLIC_GITHUB_APP_CLIENT_ID=""
    GITHUB_APP_CLIENT_SECRET=""
    GITHUB_APP_REDIRECT_URI="http://localhost:3000/api/auth/github/callback"

    # GitHub App details (will be filled after creating GitHub App)
    GITHUB_APP_NAME="open-swe-dev" # this must match the name of your GitHub app, excluding spaces
    GITHUB_APP_ID=""
    GITHUB_APP_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
    ...add your private key here...
    -----END RSA PRIVATE KEY-----
    "

    # List of GitHub usernames that are allowed to use Open SWE without providing API keys
    # This is only used in production. In development every user is an "allowed user".
    # Must be a valid JSON array of strings.
    NEXT_PUBLIC_ALLOWED_USERS_LIST='["your-github-username", "teammate-username"]'
    ```

    ### Agent Environment Variables (`apps/open-swe/.env`)

    Configure the agent environment variables:

    ```bash Environment Variables [expandable]
    # LangSmith tracing & LangGraph platform
    LANGCHAIN_PROJECT="default"
    LANGCHAIN_API_KEY="lsv2_pt_..."  # Get from LangSmith
    LANGCHAIN_TRACING_V2="true"
    LANGCHAIN_TEST_TRACKING="false"

    # LLM Provider Keys (at least one required)
    ANTHROPIC_API_KEY=""  # Recommended - default provider
    OPENAI_API_KEY=""     # Optional
    GOOGLE_API_KEY=""     # Optional

    # Infrastructure
    DAYTONA_API_KEY=""    # Required. For cloud sandboxes

    # Tools
    FIRECRAWL_API_KEY=""  # For URL content extraction

    # GitHub App settings (same as web app)
    GITHUB_APP_NAME="open-swe-dev" # this must match the name of your GitHub app, excluding spaces
    GITHUB_APP_ID=""
    GITHUB_APP_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
    ...add your private key here...
    -----END RSA PRIVATE KEY-----
    "
    GITHUB_WEBHOOK_SECRET=""  # Will be generated in next step

    # Server configuration
    PORT="2024"
    OPEN_SWE_APP_URL="http://localhost:3000"
    SECRETS_ENCRYPTION_KEY=""  # Must match web app value

    # CI/CD. See the /labs/swe/setup/ci for more information
    SKIP_CI_UNTIL_LAST_COMMIT="true"

    # List of GitHub usernames that are allowed to use Open SWE without providing API keys
    # This is only used in production. In development every user is an "allowed user".
    # Must be a valid JSON array of strings.
    NEXT_PUBLIC_ALLOWED_USERS_LIST='["your-github-username", "teammate-username"]'
    ```

    If you don't set the `NEXT_PUBLIC_ALLOWED_USERS_LIST` environment variable, every user will be required to set their own LLM API keys to use the agent. Additionally, none of the webhook features (triggering new runs by adding labels to GitHub issues, tagging the agent in PR reviews, etc.) will work unless you include the username's of the GitHub users you want to allow access to in the `NEXT_PUBLIC_ALLOWED_USERS_LIST` environment variable (in both web and agent deployments).

    <Tip>
      Generate the `SECRETS_ENCRYPTION_KEY` using: `openssl rand -hex 32`. This key must be identical in both environment files.
    </Tip>
  </Step>

  <Step title="Create GitHub App">
    <Note>
      You'll need to create a **GitHub App** (not a GitHub OAuth App). These are different types of applications with different capabilities. Consider creating separate GitHub apps for development and production environments.
    </Note>

    ### Create the GitHub App

    1. Go to [GitHub App creation page](https://github.com/settings/apps/new)
    2. Fill in the basic information:
       * **GitHub App name**: Your preferred name
       * **Description**: Development instance of Open SWE coding agent
       * **Homepage URL**: Your repository URL
       * **Callback URL**: `http://localhost:3000/api/auth/github/callback`

    ### Configure OAuth Settings

    * ✅ **Request user authorization (OAuth) during installation** - Allows users to log in to the web app
    * ✅ **Redirect on update** - Redirects users back to your app after permission updates
    * ❌ **Expire user authorization tokens** - Keep tokens from expiring

    ### Set Up Webhook

    1. ✅ **Enable webhook**

    2. **Webhook URL**: You'll need to use a tool like ngrok to expose your local server:
       ```bash
       # Install ngrok if you haven't already
       # Then expose your local LangGraph server
       ngrok http 2024
       ```
       Use the ngrok URL + `/webhooks/github` (e.g., `https://abc123.ngrok.io/webhooks/github`)

    3. **Webhook secret**: Generate and save this value:
       ```bash
       openssl rand -hex 32
       ```
       Add this value to `GITHUB_WEBHOOK_SECRET` in `apps/open-swe/.env`

    ### Configure Permissions

    **Repository permissions:**

    * **Contents**: Read & Write
    * **Issues**: Read & Write
    * **Pull requests**: Read & Write
    * **Metadata**: Read only (automatically enabled)

    **Organization permissions:** None

    **Account permissions:** None

    This gif shows how/where to enable the permissions on the GitHub App:

    <iframe className="w-full aspect-video rounded-xl" src="https://www.youtube.com/embed/rw6ddYmiTMo" title="Permissions Walkthrough" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowFullScreen />

    ### Subscribe to Events

    * ✅ **Issues** - Required for webhook functionality
    * ✅ **Pull request review** - Required for PR tagging functionality
    * ✅ **Pull request review comment** - Required for PR tagging functionality
    * ✅ **Issue comment** - Required for PR tagging functionality

    ### Installation Settings

    * **Where can this GitHub App be installed?**:
      * Choose "Any account" for broader testing
      * Or "Only on this account" to limit to your repositories

    ### Complete App Creation

    Click **Create GitHub App** to finish the setup.

    ### Collect App Credentials

    After creating the app, collect the following values and add them to both environment files:

    * **GITHUB\_APP\_NAME**: The name you chose
    * **GITHUB\_APP\_ID**: Found in the "About" section (e.g., `12345678`)
    * **NEXT\_PUBLIC\_GITHUB\_APP\_CLIENT\_ID**: Found in the "About" section
    * **GITHUB\_APP\_CLIENT\_SECRET**:
      1. Scroll to "Client secrets" section
      2. Click "Generate new client secret"
      3. Copy the generated value
    * **GITHUB\_APP\_PRIVATE\_KEY**:
      1. Scroll to "Private keys" section
      2. Click "Generate a private key"
      3. Download the `.pem` file and copy its contents
      4. Format as a single line with `\\n` for line breaks, or use the multiline format shown in the example
    * **GITHUB\_APP\_REDIRECT\_URI**: Should be `http://localhost:3000/api/auth/github/callback` for local development, or `https://your-production-url.com/api/auth/github/callback` for production.
    * **GITHUB\_WEBHOOK\_SECRET**: Generate and save this value:
      ```bash
      openssl rand -hex 32
      ```
      Add this value to `GITHUB_WEBHOOK_SECRET` in `apps/open-swe/.env` and `apps/web/.env`.

    Below are screenshots showing where in the GitHub App settings you can find the values for the environment variables.

    <Accordion title="GitHub App OAuth Settings">
      The following screenshots show where to find the following values:

      * `GITHUB_APP_ID`
      * `NEXT_PUBLIC_GITHUB_APP_CLIENT_ID`
      * `GITHUB_APP_CLIENT_SECRET`
      * `GITHUB_APP_REDIRECT_URI`
      * `GITHUB_APP_PRIVATE_KEY`

            <img src="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=fb991d989991d2caeb48ce3fbbc0a7c3" alt="GitHub Secrets Screenshot 1" width="1566" height="1270" data-path="images/gh_app_token_screenshot_1.png" srcset="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=280&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=cc5970b49d2e7ef10dd4f407d427cfef 280w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=560&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=4c45b7f451ffb0a97e92be99d440a85b 560w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=840&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=c54350d2e9ae420e7861506e85b5b883 840w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=1100&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=38ed6ead11f92edf356475720b518e03 1100w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=1650&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=0efde9106cba89a024fd78c935e73b73 1650w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=2500&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=e954ce21030c75c322a4068912c17539 2500w" data-optimize="true" data-opv="2" />
            <img src="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=c0e0c6e28a22433563c51a54170700b8" alt="GitHub Secrets Screenshot 2" width="1672" height="752" data-path="images/gh_app_token_screenshot_2.png" srcset="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=280&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=85b381cd5b4fbb3838640cf9f3c0de14 280w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=560&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=2b0ac74a8d588cc61221f3df9d11b310 560w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=840&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=139590cb21b2481512b4019993324bfc 840w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=1100&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=c6b259a0e88325fd1c56aa3748029927 1100w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=1650&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=f9e723a49554cb2f50d3b8551e2e1a74 1650w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=2500&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=e5f7675304d17b1a2426de55dbc02ab7 2500w" data-optimize="true" data-opv="2" />
            <img src="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=79ccc241a04c3ccdec140a7b2faf3746" alt="GitHub Secrets Screenshot 3" width="1652" height="656" data-path="images/gh_app_token_screenshot_3.png" srcset="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=280&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=04879f70660d41b91d1dac6ab799a863 280w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=560&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=6e75946858b954672ed6c65a8cf00d53 560w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=840&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=60f190da606dd9d0e1ef75f48d8c9ac3 840w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=1100&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=d7bf5b836e82880a8108f372c9bd101b 1100w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=1650&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=6229006d569d5c10063fb9b8e61f1c6b 1650w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=2500&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=8b0ff610c986a2201bd7d0f70711e979 2500w" data-optimize="true" data-opv="2" />
    </Accordion>

    <Tip>
      Keep your GitHub App credentials secure and never commit them to version control. The `.env` files are already included in `.gitignore`.
    </Tip>
  </Step>

  <Step title="Start Development Servers">
    With all environment variables configured, start both development servers:

    **Terminal 1 - Start the LangGraph Agent:**

    ```bash
    # apps/open-swe
    yarn dev
    ```

    This starts the LangGraph server at `http://localhost:2024`

    **Terminal 2 - Start the Web Application:**

    ```bash
    # apps/web
    yarn dev
    ```

    This starts the Next.js web app at `http://localhost:3000`

    <Note>
      Both servers need to be running simultaneously for full functionality. The web app communicates with the LangGraph agent through API calls.
    </Note>
  </Step>
</Steps>

## Verification

Once both servers are running:

1. **Visit the web app**: Navigate to `http://localhost:3000`
2. **Test GitHub authentication**: Try logging in with your GitHub account

<Tip>
  If you encounter issues, check the console logs in both terminal windows for
  error messages. Common issues include missing environment variables or
  incorrect GitHub App configuration.
</Tip>

## Next Steps

* Learn about [Authentication](/labs/swe/setup/authentication) to understand how the GitHub App integration works
* Explore [Usage](/labs/swe/usage/intro) to start using Open SWE for code changes
* Review the [Monorepo Structure](/labs/swe/setup/monorepo) for development best practices


# Development Setup

> How to set up Open SWE for development

This guide will walk you through setting up Open SWE for local development. You'll need to clone the repository, install dependencies, configure environment variables, create a GitHub App, and start the development servers.

<Note>
  This setup is for development purposes. For production deployment, you'll need
  to adjust URLs and create separate GitHub Apps for production use.
</Note>

## Prerequisites

Before starting, ensure you have the following installed:

* Node.js (version 18 or higher)
* Yarn (version 3.5.1 or higher)
* Git

## Setup Steps

<Steps>
  <Step title="Clone the Repository">
    Clone the Open SWE repository to your local machine:

    ```bash
    git clone https://github.com/langchain-ai/open-swe.git
    ```

    ```bash
    cd open-swe
    ```
  </Step>

  <Step title="Install Dependencies">
    Install all dependencies using Yarn from the repository root:

    ```bash
    yarn install
    ```

    This will install dependencies for all packages in the monorepo workspace.
  </Step>

  <Step title="Set Up Environment Files">
    Copy the environment example files and configure them:

    ```bash
    # Copy web app environment file
    cp apps/web/.env.example apps/web/.env
    ```

    ```bash
    # Copy agent environment file
    cp apps/open-swe/.env.example apps/open-swe/.env
    ```

    ### Web App Environment Variables (`apps/web/.env`)

    Fill in the following variables (GitHub App values will be added in the next step):

    ```bash Environment Variables [expandable]
    # API URLs for development
    NEXT_PUBLIC_API_URL="http://localhost:3000/api"
    LANGGRAPH_API_URL="http://localhost:2024"

    # Encryption key for secrets (generate with: openssl rand -hex 32)
    SECRETS_ENCRYPTION_KEY=""

    # GitHub App OAuth settings (will be filled after creating GitHub App)
    NEXT_PUBLIC_GITHUB_APP_CLIENT_ID=""
    GITHUB_APP_CLIENT_SECRET=""
    GITHUB_APP_REDIRECT_URI="http://localhost:3000/api/auth/github/callback"

    # GitHub App details (will be filled after creating GitHub App)
    GITHUB_APP_NAME="open-swe-dev" # this must match the name of your GitHub app, excluding spaces
    GITHUB_APP_ID=""
    GITHUB_APP_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
    ...add your private key here...
    -----END RSA PRIVATE KEY-----
    "

    # List of GitHub usernames that are allowed to use Open SWE without providing API keys
    # This is only used in production. In development every user is an "allowed user".
    # Must be a valid JSON array of strings.
    NEXT_PUBLIC_ALLOWED_USERS_LIST='["your-github-username", "teammate-username"]'
    ```

    ### Agent Environment Variables (`apps/open-swe/.env`)

    Configure the agent environment variables:

    ```bash Environment Variables [expandable]
    # LangSmith tracing & LangGraph platform
    LANGCHAIN_PROJECT="default"
    LANGCHAIN_API_KEY="lsv2_pt_..."  # Get from LangSmith
    LANGCHAIN_TRACING_V2="true"
    LANGCHAIN_TEST_TRACKING="false"

    # LLM Provider Keys (at least one required)
    ANTHROPIC_API_KEY=""  # Recommended - default provider
    OPENAI_API_KEY=""     # Optional
    GOOGLE_API_KEY=""     # Optional

    # Infrastructure
    DAYTONA_API_KEY=""    # Required. For cloud sandboxes

    # Tools
    FIRECRAWL_API_KEY=""  # For URL content extraction

    # GitHub App settings (same as web app)
    GITHUB_APP_NAME="open-swe-dev" # this must match the name of your GitHub app, excluding spaces
    GITHUB_APP_ID=""
    GITHUB_APP_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
    ...add your private key here...
    -----END RSA PRIVATE KEY-----
    "
    GITHUB_WEBHOOK_SECRET=""  # Will be generated in next step

    # Server configuration
    PORT="2024"
    OPEN_SWE_APP_URL="http://localhost:3000"
    SECRETS_ENCRYPTION_KEY=""  # Must match web app value

    # CI/CD. See the /labs/swe/setup/ci for more information
    SKIP_CI_UNTIL_LAST_COMMIT="true"

    # List of GitHub usernames that are allowed to use Open SWE without providing API keys
    # This is only used in production. In development every user is an "allowed user".
    # Must be a valid JSON array of strings.
    NEXT_PUBLIC_ALLOWED_USERS_LIST='["your-github-username", "teammate-username"]'
    ```

    If you don't set the `NEXT_PUBLIC_ALLOWED_USERS_LIST` environment variable, every user will be required to set their own LLM API keys to use the agent. Additionally, none of the webhook features (triggering new runs by adding labels to GitHub issues, tagging the agent in PR reviews, etc.) will work unless you include the username's of the GitHub users you want to allow access to in the `NEXT_PUBLIC_ALLOWED_USERS_LIST` environment variable (in both web and agent deployments).

    <Tip>
      Generate the `SECRETS_ENCRYPTION_KEY` using: `openssl rand -hex 32`. This key must be identical in both environment files.
    </Tip>
  </Step>

  <Step title="Create GitHub App">
    <Note>
      You'll need to create a **GitHub App** (not a GitHub OAuth App). These are different types of applications with different capabilities. Consider creating separate GitHub apps for development and production environments.
    </Note>

    ### Create the GitHub App

    1. Go to [GitHub App creation page](https://github.com/settings/apps/new)
    2. Fill in the basic information:
       * **GitHub App name**: Your preferred name
       * **Description**: Development instance of Open SWE coding agent
       * **Homepage URL**: Your repository URL
       * **Callback URL**: `http://localhost:3000/api/auth/github/callback`

    ### Configure OAuth Settings

    * ✅ **Request user authorization (OAuth) during installation** - Allows users to log in to the web app
    * ✅ **Redirect on update** - Redirects users back to your app after permission updates
    * ❌ **Expire user authorization tokens** - Keep tokens from expiring

    ### Set Up Webhook

    1. ✅ **Enable webhook**

    2. **Webhook URL**: You'll need to use a tool like ngrok to expose your local server:
       ```bash
       # Install ngrok if you haven't already
       # Then expose your local LangGraph server
       ngrok http 2024
       ```
       Use the ngrok URL + `/webhooks/github` (e.g., `https://abc123.ngrok.io/webhooks/github`)

    3. **Webhook secret**: Generate and save this value:
       ```bash
       openssl rand -hex 32
       ```
       Add this value to `GITHUB_WEBHOOK_SECRET` in `apps/open-swe/.env`

    ### Configure Permissions

    **Repository permissions:**

    * **Contents**: Read & Write
    * **Issues**: Read & Write
    * **Pull requests**: Read & Write
    * **Metadata**: Read only (automatically enabled)

    **Organization permissions:** None

    **Account permissions:** None

    This gif shows how/where to enable the permissions on the GitHub App:

    <iframe className="w-full aspect-video rounded-xl" src="https://www.youtube.com/embed/rw6ddYmiTMo" title="Permissions Walkthrough" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowFullScreen />

    ### Subscribe to Events

    * ✅ **Issues** - Required for webhook functionality
    * ✅ **Pull request review** - Required for PR tagging functionality
    * ✅ **Pull request review comment** - Required for PR tagging functionality
    * ✅ **Issue comment** - Required for PR tagging functionality

    ### Installation Settings

    * **Where can this GitHub App be installed?**:
      * Choose "Any account" for broader testing
      * Or "Only on this account" to limit to your repositories

    ### Complete App Creation

    Click **Create GitHub App** to finish the setup.

    ### Collect App Credentials

    After creating the app, collect the following values and add them to both environment files:

    * **GITHUB\_APP\_NAME**: The name you chose
    * **GITHUB\_APP\_ID**: Found in the "About" section (e.g., `12345678`)
    * **NEXT\_PUBLIC\_GITHUB\_APP\_CLIENT\_ID**: Found in the "About" section
    * **GITHUB\_APP\_CLIENT\_SECRET**:
      1. Scroll to "Client secrets" section
      2. Click "Generate new client secret"
      3. Copy the generated value
    * **GITHUB\_APP\_PRIVATE\_KEY**:
      1. Scroll to "Private keys" section
      2. Click "Generate a private key"
      3. Download the `.pem` file and copy its contents
      4. Format as a single line with `\\n` for line breaks, or use the multiline format shown in the example
    * **GITHUB\_APP\_REDIRECT\_URI**: Should be `http://localhost:3000/api/auth/github/callback` for local development, or `https://your-production-url.com/api/auth/github/callback` for production.
    * **GITHUB\_WEBHOOK\_SECRET**: Generate and save this value:
      ```bash
      openssl rand -hex 32
      ```
      Add this value to `GITHUB_WEBHOOK_SECRET` in `apps/open-swe/.env` and `apps/web/.env`.

    Below are screenshots showing where in the GitHub App settings you can find the values for the environment variables.

    <Accordion title="GitHub App OAuth Settings">
      The following screenshots show where to find the following values:

      * `GITHUB_APP_ID`
      * `NEXT_PUBLIC_GITHUB_APP_CLIENT_ID`
      * `GITHUB_APP_CLIENT_SECRET`
      * `GITHUB_APP_REDIRECT_URI`
      * `GITHUB_APP_PRIVATE_KEY`

            <img src="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=fb991d989991d2caeb48ce3fbbc0a7c3" alt="GitHub Secrets Screenshot 1" width="1566" height="1270" data-path="images/gh_app_token_screenshot_1.png" srcset="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=280&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=cc5970b49d2e7ef10dd4f407d427cfef 280w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=560&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=4c45b7f451ffb0a97e92be99d440a85b 560w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=840&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=c54350d2e9ae420e7861506e85b5b883 840w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=1100&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=38ed6ead11f92edf356475720b518e03 1100w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=1650&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=0efde9106cba89a024fd78c935e73b73 1650w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_1.png?w=2500&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=e954ce21030c75c322a4068912c17539 2500w" data-optimize="true" data-opv="2" />
            <img src="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=c0e0c6e28a22433563c51a54170700b8" alt="GitHub Secrets Screenshot 2" width="1672" height="752" data-path="images/gh_app_token_screenshot_2.png" srcset="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=280&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=85b381cd5b4fbb3838640cf9f3c0de14 280w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=560&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=2b0ac74a8d588cc61221f3df9d11b310 560w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=840&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=139590cb21b2481512b4019993324bfc 840w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=1100&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=c6b259a0e88325fd1c56aa3748029927 1100w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=1650&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=f9e723a49554cb2f50d3b8551e2e1a74 1650w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_2.png?w=2500&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=e5f7675304d17b1a2426de55dbc02ab7 2500w" data-optimize="true" data-opv="2" />
            <img src="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=79ccc241a04c3ccdec140a7b2faf3746" alt="GitHub Secrets Screenshot 3" width="1652" height="656" data-path="images/gh_app_token_screenshot_3.png" srcset="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=280&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=04879f70660d41b91d1dac6ab799a863 280w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=560&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=6e75946858b954672ed6c65a8cf00d53 560w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=840&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=60f190da606dd9d0e1ef75f48d8c9ac3 840w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=1100&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=d7bf5b836e82880a8108f372c9bd101b 1100w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=1650&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=6229006d569d5c10063fb9b8e61f1c6b 1650w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/gh_app_token_screenshot_3.png?w=2500&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=8b0ff610c986a2201bd7d0f70711e979 2500w" data-optimize="true" data-opv="2" />
    </Accordion>

    <Tip>
      Keep your GitHub App credentials secure and never commit them to version control. The `.env` files are already included in `.gitignore`.
    </Tip>
  </Step>

  <Step title="Start Development Servers">
    With all environment variables configured, start both development servers:

    **Terminal 1 - Start the LangGraph Agent:**

    ```bash
    # apps/open-swe
    yarn dev
    ```

    This starts the LangGraph server at `http://localhost:2024`

    **Terminal 2 - Start the Web Application:**

    ```bash
    # apps/web
    yarn dev
    ```

    This starts the Next.js web app at `http://localhost:3000`

    <Note>
      Both servers need to be running simultaneously for full functionality. The web app communicates with the LangGraph agent through API calls.
    </Note>
  </Step>
</Steps>

## Verification

Once both servers are running:

1. **Visit the web app**: Navigate to `http://localhost:3000`
2. **Test GitHub authentication**: Try logging in with your GitHub account

<Tip>
  If you encounter issues, check the console logs in both terminal windows for
  error messages. Common issues include missing environment variables or
  incorrect GitHub App configuration.
</Tip>

## Next Steps

* Learn about [Authentication](/labs/swe/setup/authentication) to understand how the GitHub App integration works
* Explore [Usage](/labs/swe/usage/intro) to start using Open SWE for code changes
* Review the [Monorepo Structure](/labs/swe/setup/monorepo) for development best practices


# CI Configuration

> How to configure your CI pipeline for Open SWE

## Skip CI until last commit

Open SWE will push a commit after every change to the repository. This will cause your GitHub CI workflows to run for every commit, which is unnecessary.

To prevent this, Open SWE supports adding an environment variable which will append `[skip ci]` to the commit message. This can be used to make GitHub skip CI for that commit.

Below are instructions showing how to use this to skip creating Vercel CI preview deployments on every commit:

<Steps>
  <Step title="Set the environment variable">
    Set the environment variable `SKIP_CI_UNTIL_LAST_COMMIT` to `true` in Open SWE's environment variables:

    `apps/open-swe/.env`

    ```bash
    SKIP_CI_UNTIL_LAST_COMMIT="true"
    ```
  </Step>

  <Step title="Add custom 'Ignored Build Step' in Vercel">
    Add a custom 'Ignored Build Step' in Vercel to skip CI when commit messages contain the string `[skip ci]`.

    1. Navigate to your Vercel project dashboard
    2. Go to **Settings** → **Git** tab
    3. Scroll to **Ignored Build Step** section
    4. Select **Custom** and add the following script:

        <img src="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/vercel_ignored_build_script_screenshot.png?fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=03207efe7b067057c310560d338425ab" alt="Vercel Custom Build Step Screenshot" width="1900" height="732" data-path="images/vercel_ignored_build_script_screenshot.png" srcset="https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/vercel_ignored_build_script_screenshot.png?w=280&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=50abd8802d45ec9a4c42bd151ec8c84b 280w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/vercel_ignored_build_script_screenshot.png?w=560&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=f07c1134c8955ead74353e3af20c648e 560w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/vercel_ignored_build_script_screenshot.png?w=840&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=c3219349b2887a4ea5df018fb37d6463 840w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/vercel_ignored_build_script_screenshot.png?w=1100&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=1526d8b9751f07c7f3ab928a1f991a4c 1100w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/vercel_ignored_build_script_screenshot.png?w=1650&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=e678b5f017118d8e9666ef1799134643 1650w, https://mintcdn.com/langchain-5e9cc07a/Xbr8HuVd9jPi6qTU/images/vercel_ignored_build_script_screenshot.png?w=2500&fit=max&auto=format&n=Xbr8HuVd9jPi6qTU&q=85&s=54181a440e97667bcaaff0ef7b5dcb37 2500w" data-optimize="true" data-opv="2" />

    ```bash
    git log -1 --pretty=oneline --abbrev-commit | grep -w "\[skip ci\]" && exit 0 || exit 1
    ```

    <Callout type="warning">
      **Do not test this command locally** as it will close your terminal. Use the testing method below instead.
    </Callout>
  </Step>

  <Step title="Test the configuration (optional)">
    To verify your setup works without closing your terminal, use this safe testing command:

    ```bash
    # Safe local testing - won't close your terminal
    if git log -1 --pretty=oneline --abbrev-commit | grep -w "\[skip ci\]"; then
        echo "Found [skip ci] - Vercel would skip this build"
    else
        echo "No [skip ci] found - Vercel would proceed with build"
    fi
    ```

    Test with both types of commits:

    * Commits with `[skip ci]` should show "Vercel would skip this build"
    * Commits without `[skip ci]` should show "Vercel would proceed with build"
  </Step>
</Steps>

### How it works

1. **During task execution**: Open SWE includes `[skip ci]` in commit messages
2. **Vercel behavior**: Skips creating preview deployments for these commits (script exits with code 1)
3. **Task completion**: Open SWE pushes a final commit without `[skip ci]` to trigger deployment (script exits with code 0)
4. **Result**: Only one preview deployment per Open SWE task
