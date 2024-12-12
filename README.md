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

## Deploying into a Kubernetes cluster

‼️ This operation was only tested at a local environment using Rancher Desktop.

Once we have the images created and in a registry, we can try to deploy the elements in a Kubernetes cluster. If images were not created previously you can do with:

```shell
docker image build . -f packages/backend/Dockerfile --tag backstage:0.0.2
docker image build . -f Dockerfile.hostbuild --tag backstage-ui:0.0.1
```

As the descriptors are defined inside the `kubernetes` folder we can follow with the commands below:

```shell
kubectl apply -f kubernetes/namespace.yaml

export AUTH_GITHUB_CLIENT_ID=$(echo -n "<client-id>" | base64)
export AUTH_GITHUB_CLIENT_SECRET=$(echo -n "<client-secret>" | base64)
envsubst < kubernetes/backstage-secrets.yaml | kubectl apply -f -

kubectl apply -f kubernetes/backstage.yaml

kubectl apply -f kubernetes/backstage-service.yaml

kubectl port-forward --namespace=backstage svc/backstage 3000:3000 7007:7007
```

Now we should be able to access our Backstage deployment at `http://localhost:3000`.
 
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
* https://backstage.io/docs/deployment/k8s
