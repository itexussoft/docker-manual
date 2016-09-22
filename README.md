# Continuous integration with Docker

In this manual we will deploy sample rails app (backed w/ PostgreSQL) on EC2 instance using
- [Circle CI](https://circleci.com/) - a continuous intgration tool,
- [Docker Hub](https://hub.docker.com/) - a registry to store docker images,
- [Docker Cloud](https://cloud.docker.com/) - a hosted service that provides a Registry with build and testing facilities for Dockerized application images.

**FYI**: [App](https://github.com/nastia-shaternik/dev-docker) that we're going to deploy on pruduction env in this session.

## Set up continuos integration with Circle CI

In this section we will link our app with Cicle CI - in order to see how
each commit integrates into the existing app.

You just need to login though your GitHub account and add a repository
you want to watch.
For plans and prices look [here](https://circleci.com/pricing/).

After that every your pushed commit will be checked via tests or via
other measurement you've provided.

## Continue with Docker Hub

Next, we want every good build to be stored at Docker Hub as an image.
Assume we have sample `Dockerfile` for our app:

```Dockerfile
FROM ruby:2.3.1
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
RUN mkdir /dev-docker
WORKDIR /dev-docker
ADD Gemfile /dev-docker/Gemfile
ADD Gemfile.lock /dev-docker/Gemfile.lock
RUN bundle install
ADD . /dev-docker
```

We need it to dockerize our app.

Please consider to [read](https://docs.docker.com/compose/rails/) how to dockerize sample rails app to use it in development because it's
not part of this guide to teach you how to work with Docker.

## Create automated build in Docker Hub

We need to create an automated build to store our image on Doker Hub.
Login to Docker Hub, link your Github or Bitbucket account in settings, and
then create automated build.

![Docker Hub link account](https://raw.githubusercontent.com/nastia-shaternik/docker-manual/master/images/docker-hub-link-gh-account.png)

![Docker Hub create automated build](https://raw.githubusercontent.com/nastia-shaternik/docker-manual/master/images/docker-hub-create-automated-build.png)

Uncheck `When active, builds will happen automatically on pushes.`
because we will control them via a CI tool.

![Docker Hub automated build settings](https://raw.githubusercontent.com/nastia-shaternik/docker-manual/master/images/docker-hub-automated-build-settings.png)

Scroll down and you will see section `Build triggers` - we need to
activate them. After activation you will get `Trigger URL`.

![Docker Hub automated build trigger](https://raw.githubusercontent.com/nastia-shaternik/docker-manual/master/images/docker-hub-activate-trigger.png)


### Push image to Docker Hub when build succeeded

To add this behaviour we need to slightly change our `cirle.yml`.

```yaml
deployment:
  dockerhub:
    branch: master
    commands:
      - $DOCKER_HUB_TRIGGER
```

As you can see here we use env variable `$DOCKER_HUB_TRIGGER`. We should
add it on Circle CI:

![CircleCI add ENV var](https://raw.githubusercontent.com/nastia-shaternik/docker-manual/master/images/circle-ci-add-env-var.png)

Insert as value: `curl -H "Content-Type: application/json" --data
'{"build": true}' -X POST #{TRIGGER_URL}`.

After these steps you should be able to see builds of images on Docker
hub after every "successful" commit.

## Linking with Docker Cloud

Docker cloud allows to
- manage nodes AKA instances which are physically hosted on another cloud providers: AWS, Digital Ocean, etc.
- manage stacks AKA different groups of services
- manage services AKA deployed containers

Now we need to link Docker Cloud with Amazon account - [read how to do
it](https://docs.docker.com/docker-cloud/infrastructure/link-aws/).

Then you will be able to create a node:

![Docker Cloud create node](https://raw.githubusercontent.com/nastia-shaternik/docker-manual/master/images/docker-cloud-create-node.png)

And provide a Stackfile:

![Docker Cloud create stack](https://raw.githubusercontent.com/nastia-shaternik/docker-manual/master/images/docker-cloud-create-stack.png)

You can click `Create` and deploy stack later or create and deploy it
immidiately.

If everything is OK you will see two containers running on your node.

Don't forget to SSH on your `web` container with terminal and create DB for rails
here:

`rake db:create db:migrate`.

![Docker Cloud service terminal](https://raw.githubusercontent.com/nastia-shaternik/docker-manual/master/images/docker-cloud-web-terminal.png)

## Redeploy our web container when new image was pushed on Docker Cloud

Firstly, you need to create new trigger on a service. Just provide a
trigger name and action that should be performed: Scale or Redeploy.

![Docker Cloud create trigger](https://raw.githubusercontent.com/nastia-shaternik/docker-manual/master/images/docker-cloud-redeploy-trigger.png)

Then create a webhook for docker hub image:

![Docker Hub web hook](https://raw.githubusercontent.com/nastia-shaternik/docker-manual/master/images/docker-hub-web-hook.png)

And that's all! Pretty simple.

## Conclusion

It's the end of this very concise guide and I suppose a lot of points
aren't covered deeply here.

If you have any questions - please share them as issues.
Thanx!
