# syntax=docker/dockerfile:1

ARG NODE_VERSION=16

################################################################################
# Use node image for base image for all stages.
FROM node:${NODE_VERSION}-alpine as base

# Set working directory for all build stages.
WORKDIR /usr/src/app

################################################################################
# Create a stage for building the application.
FROM base as build

# Download additional development dependencies before building, as some projects require
RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=package-lock.json,target=package-lock.json \
    --mount=type=cache,target=/root/.npm \
    npm ci

# Copy the rest of the source files into the image.
COPY . .
# Run the build script.
RUN npm run build

################################################################################
# Create a new stage to run the application with minimal runtime dependencies
# where the necessary files are copied from the build stage.
FROM base as final

# Use production node environment by default.
ENV NODE_ENV production

# Install serve to expose application
RUN npm install -g serve@14.2.1

# Run the application as a non-root user.
USER node

# Copy package.json so that package manager commands can be used.
COPY package.json .

# Copy the built application from the build stage into the image.
COPY --from=build /usr/src/app/build ./build

# Expose the port that the application listens on.
EXPOSE 5000

# Run the application.
ENTRYPOINT ["serve", "-s", "-l", "5000", "build"]
