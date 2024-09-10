<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://github.com/daytonaio/daytona/raw/main/assets/images/Daytona-logotype-white.png">
    <img alt="Daytona logo" src="https://github.com/daytonaio/daytona/raw/main/assets/images/Daytona-logotype-black.png" width="40%">
  </picture>
</div>

<br />

<div align="center">

[![Issues - daytona](https://img.shields.io/github/issues/daytonaio/backstage-plugins)](https://github.com/daytonaio/backstage-plugins/issues)

</div>

<h1 align="center">Daytona Backstage Plugins</h1>

<div align="center">
This repository is the home of the <a href="https://github.com/daytonaio/daytona">Daytona</a> Backstage Plugins.
</div>

</br>

<p align="center">
  <a href="https://github.com/daytonaio/backstage-plugins/issues/new?assignees=&labels=bug&projects=&template=bug_report.md&title=%F0%9F%90%9B+Bug+Report%3A+">Report Bug</a>
    ·
  <a href="https://github.com/daytonaio/backstage-plugins/issues/new?assignees=&labels=enhancement&projects=&template=feature_request.md&title=%F0%9F%9A%80+Feature%3A+">Request Feature</a>
    ·
  <a href="https://go.daytona.io/slack">Join Our Slack</a>
    ·
  <a href="https://twitter.com/Daytonaio">Twitter</a>
</p>

## Plugins Overview

To integrate Daytona plugins into your existing Backstage instance, install the following plugins:

1. [@daytonaio/backstage-plugin-auth-backend-module-daytona-provider](https://www.npmjs.com/package/@daytonaio/backstage-plugin-auth-backend-module-daytona-provider)
2. [@daytonaio/daytona-web](https://www.npmjs.com/package/@daytonaio/daytona-web)
3. [@daytonaio/backstage-plugin-daytona](https://www.npmjs.com/package/@daytonaio/backstage-plugin-daytona)

### Daytona Backstage Auth Backend Plugin

The package provides Backstage module to implement authentication via `auth` plugin. It uses OIDC for authentication with Daytona Keycloak to directly authenticate the user from Daytona and login with any of the two common sign-in resolvers.

- `emailMatchingUserEntityProfileEmail`: Match the user email with the Backstage user profile email.
- `emailLocalPartMatchingUserEntityName`: Match the local part of the user email with the Backstage user profile email. Suppose, if the user email is *example@daytonaio*, then a Backstage user profile with the name example must exist.

#### Configuration

The Daytona module configuration will be under auth in app-config.yaml:

```yaml
auth:
  session:
    secret: dummy
  environment: production
  # see https://backstage.io/docs/auth/ to learn about auth providers
  providers:
    # See https://backstage.io/docs/auth/guest/provider
    daytona:
      production:
        clientId: <daytona-client-id>
        clientSecret: <daytona-client-secret>
        metadataUrl: https://id.<your-daytona-domain>/realms/default/.well-known/openid-configuration
        prompt: auto
        signIn:
          resolvers:
            - resolver: emailLocalPartMatchingUserEntityName
```

In the above configuration, these are the options required to be handled:

- `auth.session.secret`: Must be provided as required in Backstage authentication.
- Add `daytona` under the `auth.providers` section, where multiple environments can be configured and then referred in `auth.environment`.
- Add your Daytona Backstage client ID and secret. If you are using the provided `backstage.json`, then your client ID will be backstage in default `realm` and secret can be fetched by logging to `https://id.<your-daytona-domain>`.
- Sign-in resolvers can be as provided, either `emailMatchingUserEntityProfileEmail` or `emailLocalPartMatchingUserEntityName`.

#### Installation

The plugin can be installed by running the below command in Backstage root directory and later, adding the following in your Backstage backend.

```sh
# From your Backstage root directory
yarn --cwd packages/backend add @daytonaio/backstage-plugin-auth-backend-module-daytona-provider
```

#### Setup

```typescript
// In packages/backend/src/index.ts

// Add the Daytona auth plugin provider
backend.add(import('@daytonaio/backstage-plugin-auth-backend-module-daytona-provider'));
```

### Daytona Backstage Plugin

The Daytona plugin provides frontend components to connect to the Daytona API backend and view the workspaces for the authenticated user. You can create new Daytona workspaces directly from Backstage.

#### Features

- Authenticates users with Daytona Keycloak.
- Displays a list of all workspaces for the authenticated user.
- Shows workspaces of a specific repository with proper annotations.
- Allows users to create new workspaces by navigating to the Daytona instance.

#### Installation

The package shall be installed in the Backstage root directory as below.

```sh
yarn --cwd packages/app add @daytonaio/backstage-plugin-daytona
```

#### Setup

1. Install the plugin dependency in your Backstage app package:

```sh
# From your Backstage root directory
yarn add --cwd packages/app @daytonaio/backstage-plugin-daytona
```

2. Add to the app `EntityPage` component. Make sure to add `DaytonaOverviewComponent` right after `EntityAboutCard` under `overviewContent`. This will get the repository URL automatically from the entity location metadata to create the Daytona Workspaces. Along with that, it will also list all the Workspaces, specific to the repository.

```typescript
// In packages/app/src/components/catalog/EntityPage.tsx
import { DaytonaOverviewContent } from '@daytonaio/backstage-plugin-daytona';

// Add the DaytonaOverviewContent to show the workspaces for that entity
const overviewContent = (
  <Grid container spacing={3} alignItems="stretch">
    <Grid item md={6}>
      <EntityAboutCard variant="gridItem" />
    </Grid>
    <Grid item md={6}>
      <DaytonaOverviewContent />
    </Grid>
    {/* other grid items here*/}
  </Grid>
);
```

3. Annotate your component with a valid Git repository if you wish to override the automatically configured repository URL for creating Daytona Workspaces.

The annotation key is `daytona.io/repo-url`.

Example:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: backstage
  description: backstage.io
  annotations:
    daytona.io/repo-url: https://github.com/daytonaio-templates/go
spec:
  type: website
  lifecycle: production
  owner: user:guest
```

##### App Menu Requirements

Ensure that the package is installed as mentioned in the Installation section.

1. Add the following code snippet to the `App.tsx` component of your application:

```typescript
// In packages/app/src/App.tsx
import { DaytonaPage } from '@daytonaio/backstage-plugin-daytona';

// Add the route to the App path routes
const routes = (
  <FlatRoutes>
  {/* other routes here */}
      <Route path="/daytona" element={<DaytonaPage />} />
  </FlatRoutes>
);
```

2. Add the following code snippet to the `Root.tsx` component of your application:

```typescript
// In packages/app/src/components/Root/Root.tsx
import { DaytonaIcon } from '@daytonaio/backstage-plugin-daytona';

// Add the menu to the Root menu sidebar
export const Root = ({ children }: PropsWithChildren<{}>) => (
  <SidebarPage>
    <Sidebar>
    {/* other sidebar items here */}
    {/* add inside "Menu" SidebarGroup */}
      <SidebarItem icon={DaytonaIcon} to="daytona" text="Daytona" />
    {/* other sidebar items here */}
    </Sidebar>
    {children}
  </SidebarPage>
);
```

#### Configuration

This plugin requires the domain URL for your Daytona instance. This can be configured in `app-config.yaml` file as per below snippet:

```yaml
daytona:
  domain: daytona.domain.com
```

##### CORS Configuration for connecting Daytona APIs with Backstage

Below configurations need to be added in the `watkins` ingress YAML which allows CORS connectivity. It allows Daytona URL along with Backstage URL for CORS policy.

```yaml
nginx.ingress.kubernetes.io/cors-allow-credentials: "true"
nginx.ingress.kubernetes.io/cors-allow-headers: Authorization, Access-Control-Allow-Origin, Access-Control-Allow-Headers, Origin, Content-Type, Accept, X-Requested-With
nginx.ingress.kubernetes.io/cors-allow-methods: PUT, GET, POST, OPTIONS, DELETE
nginx.ingress.kubernetes.io/cors-allow-origin: https://<backstage-app-url>, https://<daytona-domain-url>
nginx.ingress.kubernetes.io/enable-cors: "true"
```

#### Setup Backstage Auth with Keycloak

Backstage shall be registered as a Keycloak client in the default realm. Once the client is created, client ID and secret can be configured in the Backstage instance.

1. Create Keycloak client in the `default` realm using `backstage.json` file.
2. Register the newly created client for authentication in backstage.

```yaml
auth:
 environment: production
 providers:
   daytona:
     production:
       clientId: <daytona-client-id>
       clientSecret: <daytona-client-secret>
       metadataUrl: https://id.<daytona-domain-url>/realms/default/.well-known/openid-configuration
       prompt: auto
```

### Daytona Web Plugin

The plugin provides the frontend components required for Daytona Authentication (`daytonaApiFactory`) and Sign-In (`daytonaSignInProvider`). These components will ease the implementation for Daytona authentication using OAuth in Backstage.

#### Features

- Provides `daytonaApiFactory` for Daytona authentication.
- Includes `daytonaSignInProvider` to support Daytona OAuth sign-in.

#### Installation

Install the package via Yarn in your Backstage root directory:

```sh
# From your Backstage root directory
yarn --cwd packages/app add @daytonaio/daytona-web
```

#### Authentication Setup

Backstage requires ApiFactory to interact with Daytona OAuth library and a sign-in provider. Follow the below steps:

1. In Backstage folder `packages/app/src`, add the below snippet in `apis.ts` file.

```typescript
// In packages/app/src/apis.ts
import { daytonaApiFactory } from '@daytonaio/daytona-web';

// Add the Daytona ApiFactory to the list of available APIs
export const apis: AnyApiFactory[] = [
    {/* other ApiFactory here */}
    daytonaApiFactory
];
```

2. Add the following to Backstage `App.tsx` file.

```typescript
// In packages/app/src/App.tsx
import { daytonaSignInProvider } from '@daytonaio/daytona-web';

// Add the Daytona Sign-In Provider to the available sign-in providers
const app = createApp({
    {/* other api, bind routes here */}
    components: {
        SignInPage: props => <SignInPage {...props} auto providers={['guest',daytonaSignInProvider]} />,
    },
});
```
