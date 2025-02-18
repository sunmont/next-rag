#FROM node:22.13.1-alpine AS base
FROM node:alpine AS base

# Set the working directory inside the container to /app
WORKDIR /app

# Copy package.json and package-lock.json files into the working directory
COPY package*.json ./
RUN npm install
COPY . .

FROM base AS dev
ENV CHOKIDAR_USEPOLLING=true
ENV WATCHPACK_POLLING=true
CMD ["npm", "run", "db:migrate"]

FROM base AS build
WORKDIR /app
ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

FROM base AS prod
WORKDIR /app

ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=build /app/public ./public
RUN mkdir .next
RUN chown nextjs:nodejs .next

COPY --from=build --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=build --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["npm", "run", "start:migrate:prod"]
