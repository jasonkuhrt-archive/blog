# Multi-Stage Docker Builds

2018 Feb 27

[Docker multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) are a mechanism to orchestrate docker builds that optimize for the smallest possible resulting image size without resorting to complicated external scriping.

An early (July 2017) Docker blog post about multi-stage builds can be read [here](https://blog.docker.com/2017/07/multi-stage-builds/).

The following is what I currently use to build my node apps for production. It might need tweaks to serve larger projects (e.g. Alpine base might be an issue for native npm dependencies).

```docker
FROM node:8.1-alpine AS base
WORKDIR /app
COPY package.json yarn.lock .npmrc ./
RUN yarn --pure-lockfile --production

FROM base AS test
COPY . /app/
RUN yarn --pure-lockfile
RUN yarn run test

FROM test AS build
COPY . /app/
RUN yarn run build

FROM base AS release
COPY --from=build /app/build /app/build/
ENV NODE_ENV production
EXPOSE 80
ENTRYPOINT ["yarn", "run"]
CMD ["start"]
```

The overall flow works like this:

1. Build a production-oriented base.

2. Build a test-oriented version which is a superset of production. Run tests here. This image has all development dependencies.

3. Build the app based off the test image because building the app requires development dependencies. This step could take place within the test image but is split out for clarity.

4. Build the final production release image. Its based off the base image which contains just the production dependencies and three more files. We copy the built app from the `build` image into the `release` image. Then we're done!

Docker multi-stage builds can shave hundreds of megabytes off final app images when there are many development dependencies which in node happens all the time with so much supporting tooling e.g. `babel`, `jest`, `webpack`, `pretter`, etc.

Its interesting to me how the test suite now runs co-located in the build whereas before it typically lives in the CI configuration e.g. `.travis.yml`.

An area for future investigation is combining multi-stage docker builds with `ONBUILD` to create a reusable build pipeline. Busbud blogged about some of [their explorations in this area](https://engineering.busbud.com/2017/10/09/node-interactive-2017/).
