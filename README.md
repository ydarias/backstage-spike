# [Backstage](https://backstage.io)

This is your newly scaffolded Backstage App, Good Luck!

To start the app, run:

```sh
yarn install
yarn dev
```

## Deploying with Docker images

The recommended way to do this is using the so called Host Build.

This means that most part of the building process is done outside a Docker image, which is faster and handles better the caches. To do that we just can execute the following commands:

```shell
yarn install --immutable

# tsc outputs type definitions to dist-types/ in the repo root, which are then consumed by the build
yarn tsc

# Build the backend, which bundles it all up into the packages/backend/dist folder.
yarn build:backend
```

Now we can create the Docker image with `yarn build-image` but the documentation also mentions:

```shell
docker image build . -f packages/backend/Dockerfile --tag backstage
```

And finally we can run with:

```shell
docker run -it -p 7007:7007 backstage
```

### Important development notes

Doing the previous steps, we already had some problems, so a few tips here.

Because the `guest` authentication method is disabled at production by default, it fails if no other auth method is configured. The recommended option is to use Github Authentication system, but there are other options available. 

Once we have the authentication running we can run with:

```shell
docker run -it -p 7007:7007 -e AUTH_GITHUB_CLIENT_ID='<client-id>' -e AUTH_GITHUB_CLIENT_SECRET='<client-secret>' backstage
```

The second problem was that the embedded front served by the backed was not working properly not being able to get the data behind some authentication problems. So the option selected was to run the frontend separately. During development test it is enough to execute `yarn start`, but we will have to discover how to create a Docker image for the frontend.

### Creating frontend image

To create the frontend image we can execute the following commands at the project root level:

```shell
yarn install --immutable

yarn tsc

yarn workspace app build --config ../../app-config.yaml --config ../../app-config.production.yaml

docker build -t backstage-frontend -f Dockerfile.hostbuild .
```

We can ran the Docker image as:

```shell
docker run -it -p 3000:80 backstage-frontend
```

## Technology Radar plugin

Backstage offers a multitude of plugins, and one of them allows to have a Technology Radar a la ThoughtWorks.

### Installation and configuration

First, we need to install two plugins to make it work.

```shell
yarn --cwd packages/app add @backstage-community/plugin-tech-radar
```

Add the route to the `packages/app/src/App.tsx` file:

```typescript jsx
const routes = (
  <FloatRoutes>
    ...
    <Route
      path="/tech-radar"
      element={<TechRadarPage width={1500} height={800} />}
    />
  </FloatRoutes>
)
```

Also, you need to anchor a link to the sidebar, so change the `packages/app/src/components/Root/Root.tsx` file as:

```typescript jsx
export const Root = ({ children }: PropsWithChildren<{}>) => (
  <SidebarPage>
    <Sidebar>
      ...
      <SidebarGroup label="Menu" icon={<MenuIcon />}>
        ...
        <SidebarItem icon={HubIcon} to="tech-radar" text="Tech Radar" />
      </SidebarGroup>
    </Sidebar>
  </SidebarPage>
);
```

Now we want to be able to have our own data definition for the Technology Radar, so we will use a tech-radar-backend plugin for that.

```shell
yarn --cwd packages/backend add @backstage-community/plugin-tech-radar-backend
```

![Backstage Tech Radar components diagram](docs/assets/backstage-tech-radar.png)

Using the `tech-radar-backend` plugin we are able to define at the configuration a URL where the JSON lives. So we can keep the technology radar content in a different repo or system, just referred at `app-config.yaml` with a block like:


```yaml
techRadar:
  url: https://github.com/ydarias/backstage-spike/blob/main/public/static/techRadarData.json
```

**Note**: Sometimes it takes a few minutes to reload the changes, give it a time before determining it is not working.

## Interesting links and documentation

* https://github.com/backstage/community-plugins/tree/main/workspaces/tech-radar/plugins/tech-radar
* https://github.com/backstage/community-plugins/blob/main/workspaces/tech-radar/plugins/tech-radar-backend
* https://backstage.io/docs/getting-started/config/authentication
