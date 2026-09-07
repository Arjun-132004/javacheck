FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN if [ -f package-lock.json ]; then npm ci; else npm install; fi
COPY . .
# Build-time env for the frontend build (added by DeepAgent).
# These are framework-prefixed, i.e. inlined into the public bundle by
# design — never put a credential in a build arg. The empty defaults keep
# the image buildable with no --build-arg; CI overrides them with the
# real values.
ARG VITE_API_URL=""
ENV VITE_API_URL=${VITE_API_URL}
ARG VITE_BACKEND_UR=""
ENV VITE_BACKEND_UR=${VITE_BACKEND_UR}
ARG VITE_FRONTEND_URL=""
ENV VITE_FRONTEND_URL=${VITE_FRONTEND_URL}
RUN npm run build

# Unprivileged nginx: uid 101, listens on 8080, writable paths under /tmp.
# Stock nginx here would need root for :80 and /var/cache/nginx and exits
# [emerg] under the manifest's runAsNonRoot + readOnlyRootFilesystem.
FROM nginxinc/nginx-unprivileged:1.27-alpine
COPY --from=build /app/dist /usr/share/nginx/html
# Already the image's default; stated so the manifest's runAsUser has
# something to agree with and the review rules can see it.
USER 101
EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
