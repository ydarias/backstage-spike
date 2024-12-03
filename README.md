# [Backstage](https://backstage.io)

This is your newly scaffolded Backstage App, Good Luck!

To start the app, run:

```sh
yarn install
yarn dev
```

## Technology Radar plugin

Backstage offers a multitude of plugins, and one of them allows to have a Technology Radar a la ThoughtWorks.

### Installation and configuration

First, we need to install two plugins to make it work.

```shell
yarn --cwd packages/app add @backstage-community/plugin-tech-radar
```

```shell
yarn --cwd packages/backend add @backstage-community/plugin-tech-radar-backend
```

![Backstage Tech Radar components diagram](docs/assets/backstage-tech-radar.png)

Using the `tech-radar-backend` plugin we are able to define at the configuration a URL where the JSON lives. So we can keep the technology radar content in a different repo or system, just referred at `app-config.yaml` with a block like:


```yaml
techRadar:
  url: https://github.com/ydarias/backstage-spike/blob/main/public/static/techRadarData.json
```

## Interesting links and documentation

* https://github.com/backstage/community-plugins/tree/main/workspaces/tech-radar/plugins/tech-radar
* https://github.com/backstage/community-plugins/blob/main/workspaces/tech-radar/plugins/tech-radar-backend
