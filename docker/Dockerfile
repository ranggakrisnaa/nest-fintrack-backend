##################
# BUILD BASE IMAGE
##################

FROM node:20-alpine AS base

# Install and use pnpm
RUN npm install -g pnpm

#############################
# BUILD FOR LOCAL DEVELOPMENT
#############################

FROM base AS development
WORKDIR /app

# Copy package files first for better caching
COPY package*.json pnpm-lock.yaml ./

# Install all dependencies (including devDependencies)
RUN pnpm install

# Bundle app source
COPY . .

# Use the node user from the image (instead of the root user)
USER node

#####################
# BUILD BUILDER IMAGE
#####################

FROM base AS builder
WORKDIR /app

# Copy package files and install dependencies
COPY package*.json pnpm-lock.yaml ./
COPY --from=development /app/node_modules /app/node_modules
COPY --from=development /app/src /app/src
COPY --from=development /app/tsconfig.json /app/tsconfig.json
COPY --from=development /app/tsconfig.build.json /app/tsconfig.build.json
COPY --from=development /app/nest-cli.json /app/nest-cli.json

RUN pnpm build

# Removes unnecessary packages and re-install only production dependencies
ENV NODE_ENV production
RUN pnpm prune --prod
RUN pnpm install --prod

USER node

######################
# BUILD FOR PRODUCTION
######################

FROM node:20-alpine AS production
WORKDIR /app

# Create directories for generated files
RUN mkdir -p src/generated && chown -R node:node src

# Copy the bundled code from the build stage to the production image
COPY --from=builder /app/src/generated/i18n.generated.ts ./src/generated/i18n.generated.ts
COPY --from=builder /app/node_modules /app/node_modules
COPY --from=builder /app/dist /app/dist
COPY --from=builder /app/package.json /app/package.json

USER node

# Start the server using the production build
CMD [ "node", "dist/main.js" ]
