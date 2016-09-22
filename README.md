# Continuous integration with Docker

In this manual we will deploy sample rails app (backed w/ PostgreSQL) on EC2 instance using [Circle CI](https://circleci.com/), [Docker Hub](https://hub.docker.com/), [Docker Cloud](https://cloud.docker.com/).

FYI: [App](https://github.com/nastia-shaternik/dev-docker) that we're going to use in this session.

# Set up continuos integration with Circle CI

In this section we will link our app with Cicle CI - in order to see how
each commit integrates into the existing app.

You just need to 
