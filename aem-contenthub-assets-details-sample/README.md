# SampleApp

Welcome to my Adobe I/O Application!

## Setup

- Populate the `.env` file in the project root and fill it as shown [below](#env)

## Local Dev

- `aio app run` to start your local Dev server
- App will run on `localhost:9080` by default

By default the UI will be served locally but actions will be deployed and served from Adobe I/O Runtime. To start a
local serverless stack and also run your actions locally use the `aio app run --local` option.

## Test & Coverage

- Run `aio app test` to run unit tests for ui and actions
- Run `aio app test --e2e` to run e2e tests

## Deploy & Cleanup

- `aio app deploy` to build and deploy all actions on Runtime and static files to CDN
- `aio app undeploy` to undeploy the app

## Config

### `.env`

You can generate this file using the command `aio app use`. 

```bash
# This file must **not** be committed to source control

## please provide your Adobe I/O Runtime credentials
# AIO_RUNTIME_AUTH=
# AIO_RUNTIME_NAMESPACE=
```

### `app.config.yaml`

- Main configuration file that defines an application's implementation. 
- More information on this file, application configuration, and extension configuration 
  can be found [here](https://developer.adobe.com/app-builder/docs/guides/appbuilder-configuration/#appconfigyaml)

#### Action Dependencies

- You have two options to resolve your actions' dependencies:

  1. **Packaged action file**: Add your action's dependencies to the root
   `package.json` and install them using `npm install`. Then set the `function`
   field in `app.config.yaml` to point to the **entry file** of your action
   folder. We will use `webpack` to package your code and dependencies into a
   single minified js file. The action will then be deployed as a single file.
   Use this method if you want to reduce the size of your actions.

  2. **Zipped action folder**: In the folder containing the action code add a
     `package.json` with the action's dependencies. Then set the `function`
     field in `app.config.yaml` to point to the **folder** of that action. We will
     install the required dependencies within that directory and zip the folder
     before deploying it as a zipped action. Use this method if you want to keep
     your action's dependencies separated.

## Debugging in VS Code

While running your local server (`aio app run`), both UI and actions can be debugged, to do so open the vscode debugger
and select the debugging configuration called `WebAndActions`.
Alternatively, there are also debug configs for only UI and each separate action.

## Typescript support for UI

To use typescript use `.tsx` extension for react components and add a `tsconfig.json` 
and make sure you have the below config added
```
 {
  "compilerOptions": {
      "jsx": "react"
    }
  } 
```

## Workfront Integration

This sample adds a Content Hub side panel that can create a Workfront task and link the currently selected asset to that task.

- **UI panel**: `src/aem-contenthub-assets-details-1/web-src/src/components/PanelWorkfrontExtensionTab.js`
- **Backend action**: `src/aem-contenthub-assets-details-1/actions/generic/index.js`

### Prerequisites

**App Builder & Local Setup**

- Access to Adobe Developer Console in the correct IMS organization
- App Builder entitlement for your org
- Node.js and npm installed locally
- Adobe I/O CLI (aio) installed globally
- A GitHub Personal Access Token (required when initializing from Content Hub sample repository)
- Familiarity with JavaScript/TypeScript, React, and basic REST APIs

**Content Hub UI Extension**

- Content Hub enabled for your AEM as a Cloud Service environment
- Permissions to access Content Hub

**Workfront Integration**

- A Workfront environment
- A technical integration or API client capable of server-side authentication
- Permissions to create tasks and link external documents
- A Workfront admin to configure the document provider for AEM/Content Hub

### Step-by-Step Setup

1. To enable secure server-to-server communication between your Content Hub extension, Adobe I/O Runtime, and Workfront, configure IMS Technical Account.
   - Go to Admin Console.
   - Navigate to Products → Workfront → Workfront link.
   - Add the Technical Account as both User and Admin.
   - Ensure your own user is also added as both User and Admin in Admin Console and in Workfront.
2. In Adobe Developer Console, create or verify a Server-to-Server (JWT) integration for Workfront.
   Collect the following values from Service Credentials:
   - `IMS_ENDPOINT`
   - `METASCOPES`
   - `TECHNICAL_ACCOUNT_CLIENT_ID`
   - `TECHNICAL_ACCOUNT_CLIENT_SECRET`
   - `TECHNICAL_ACCOUNT_EMAIL`
   - `TECHNICAL_ACCOUNT_ID`
   - `ORGANIZATION_ID`
   - `PRIVATE_KEY`
   - `PUBLIC_KEY`
3. In Workfront, confirm API access and required permissions.
   - Note your tenant base URL for `WORKFRONT_BASE_URL` (e.g., `https://<company>.my.workfront.com/attask/api/v15.0`).
   - Ensure a default project exists and note its `DEFAULT_PROJECT_ID`.
   - If using AEM external documents, get the `DOCUMENT_PROVIDER_ID` from Setup → Documents → External Document Providers.
   - Ensure permissions to create tasks in the default project and to link external documents.
   - Get Workfront user ID (used by DOCUMENT_PROVIDER_ID API when needed):

     ```bash
     curl --location 'https://<your-tenant>.my.workfront.com/attask/api-internal/user/realUser' \
       --header 'Authorization: Bearer <ACCESS_TOKEN>'
     # Response → use ID field as USER_ID in DOCUMENT_PROVIDER_ID API
     ```

   - One-time creation of DOCUMENT_PROVIDER_ID via API (optional):

     ```bash
     curl --location --request PUT \
       "https://<your-tenant>.my.workfront.com/attask/api/unsupported/user/<USER_ID>?action=initializeStatelessDocumentProviderForUser" \
       --header 'user-agent: Workfront Fusion/production' \
       --header 'content-type: application/json' \
       --header 'authorization: Bearer <ACCESS_TOKEN>' \
       --data '{
         "providerType": "AEM",
         "documentProviderConfigID": "<ACTIVE_AEM_INTEGRATION_CONFIG_ID>",
         "documentProviderConfigName": "Content Hub"
       }'
     ```

     - Replace `<USER_ID>` with the Workfront user ID.
     - Replace `<ACTIVE_AEM_INTEGRATION_CONFIG_ID>` with the ID of the active AEM provider configuration in Workfront.
     - The resulting configuration ID is what you set as `DOCUMENT_PROVIDER_ID` in your `.env`.
4. All secrets must be added to your App Builder workspace and provided via `.env`.
   These are read by the Runtime action via
   `ext.config.yaml -> runtimeManifest.packages.aem-contenthub-assets-details-1.actions.generic.inputs`.

   In Stage and Production, provide all secrets via `.env` files so that `aio` can inject them at build and deploy time.

   **Required Environment Variables**

   ```bash

   # IMS / Technical Account
   IMS_ENDPOINT=ims-na1.adobelogin.com
   METASCOPES=<comma-separated Workfront metascopes>
   TECHNICAL_ACCOUNT_CLIENT_ID=<client_id>
   TECHNICAL_ACCOUNT_CLIENT_SECRET=<client_secret>
   TECHNICAL_ACCOUNT_EMAIL=<tech_account_email>
   TECHNICAL_ACCOUNT_ID=<tech_account_id>
   ORGANIZATION_ID=<org_id>
   PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
   PUBLIC_KEY="-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----\n"
   CERTIFICATE_EXPIRATION_DATE=<optional>

   # Workfront
   WORKFRONT_BASE_URL=https://<your-workfront-domain>.my.workfront.com/attask/api/v15.0
   AEM_AUTHOR=<your-aem-author-host>
   DOCUMENT_PROVIDER_ID=<provider_id_configured_in_workfront>
   DEFAULT_PROJECT_ID=<target_project_id_for_new_tasks>

   # Runtime Namespace
   AIO_runtime_namespace=<your_runtime_namespace>
   ```

   Verify namespace:

   ```bash
   aio runtime namespace get
   ```

   When selecting a workspace using the Adobe I/O CLI, the namespace is automatically populated in `.env`.

5. Edit `src/aem-contenthub-assets-details-1/web-src/src/components/ExtensionRegistration.js` and update `allowedRepos` to include your Delivery host.
6. Run locally with `aio app run` and verify the Workfront panel in Content Hub. Deploy using `aio app deploy`.

### Why Runtime Namespace Is Required

- The UI constructs the backend URL using the namespace to reach your deployed web action, e.g. `https://<AIO_runtime_namespace>.adobeio-static.net/api/v1/web/aem-contenthub-assets-details-1/generic`.
- This ensures requests route to the correct Adobe I/O Runtime workspace (dev/stage/prod). If the namespace is wrong or missing, calls 404 or hit the wrong environment.
- You can see this used in `src/aem-contenthub-assets-details-1/web-src/src/components/PanelWorkfrontExtensionTab.js` where `backendUrl` is built from `process.env.AIO_runtime_namespace`.
- Ensure your Workfront API version in `WORKFRONT_BASE_URL` matches your tenant (the example uses `v15.0`).

### How Flow Works

1. The UI tab gathers Task Name/Description and calls the web action with `action: "createTaskAndLinkAsset"` and the current asset ID.
2. The action exchanges a JWT for an IMS access token using your integration.
3. It creates a Workfront task (project = `DEFAULT_PROJECT_ID`).
4. It links the current asset to that task using `DOCUMENT_PROVIDER_ID` and `AEM_AUTHOR`.

### Troubleshooting

- **Missing env vars**: The action will return 500 with a message listing missing configuration elements.
- **401/403 from Workfront**: Verify `METASCOPES`, IMS integration credentials, and `WORKFRONT_BASE_URL`.
- **Unknown action**: Ensure the UI sends `action: createTaskAndLinkAsset` and your action is deployed.
- **Panel hidden**: Make sure your repo host is in `allowedRepos` in `ExtensionRegistration.js`.
