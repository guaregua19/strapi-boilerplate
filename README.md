# Strapi + TypeScript + Docker + PostgreSQL Boilerplate

This repository provides a robust boilerplate to kickstart your full-stack development with Strapi, TypeScript, Docker, and PostgreSQL. It includes a comprehensive setup with all the necessary configurations to streamline your development process and ensure scalability and maintainability.

## Features

- **Strapi Integration**: Leverage the powerful headless CMS capabilities of Strapi for building APIs effortlessly.
- **TypeScript Support**: Benefit from static typing and enhanced code quality with TypeScript.
- **Dockerized Setup**: Easily manage and deploy your application using Docker, ensuring a consistent environment.
- **PostgreSQL Database**: Utilize PostgreSQL for a reliable and scalable relational database management system.
- **Health Check**: Built-in health check for monitoring the application's status.
- **Environment Variables**: Centralized environment configuration using `.env` file.

## Getting Started

### Prerequisites

- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/install/)

### Installation

1. **Clone the repository**:

   ```sh
   git clone https://github.com/your-username/your-repo-name.git
   cd your-repo-name
   ```

2. **Create a `.env` file**:

   ```sh
   cp .env.example .env
   ```

   Edit the `.env` file to match your configuration.

3. **Build and start the application**:

   ```sh
   docker-compose up --build
   ```

4. **Access the Strapi Admin Panel**:
   Open your browser and navigate to `http://localhost:1337/admin`.

## Docker Configuration

### Dockerfile.local

This file builds our Strapi application's Docker image based on Node.js, specifically using an Alpine Linux variant for a lightweight environment. It installs necessary system dependencies like `vips-dev` for image processing, which Strapi often requires, especially for media handling. The Dockerfile also handles the installation of NPM packages and builds the Strapi project.

```Dockerfile
# We use node18-alpine3.17 because we must avoid python version 3.11 as it creates issues with sharp library which is needed for file upload
# https://forum.strapi.io/t/facing-error-on-installation-of-strapi-something-went-wrong-installing-the-sharp-module/16420/29
FROM node:18-alpine3.17

# Installing libvips-dev for sharp Compatibility
RUN apk update && apk add --no-cache build-base gcc autoconf automake zlib-dev libpng-dev nasm bash vips-dev

ARG NODE_ENV=development
ENV NODE_ENV=${NODE_ENV}

WORKDIR /opt/

COPY ./package.json ./
COPY ./package-lock.json ./

ENV PATH /opt/node_modules/.bin:$PATH

RUN npm install --platform=linux --arch=x64 sharp
RUN npm install

# Install the pg module
RUN npm install pg --save

WORKDIR /opt/app

COPY ./ .

RUN npm run build

EXPOSE 1337

CMD ["npm", "run", "develop"]
```

### Docker Compose Setup

In our `docker-compose.yml`, we define the services required for our application: the Strapi service and a PostgreSQL service. Docker Compose allows us to manage multiple containers as a single application.

```yaml
version: "3"
services:
  strapi-app:
    container_name: strapi-app
    build:
      context: .
      dockerfile: Dockerfile.local
    image: strapi:latest
    restart: unless-stopped
    extra_hosts:
      - "host.docker.internal:host-gateway"
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "wget --spider --quiet --tries=1 --timeout=5 http://localhost:1337/_health || exit 1",
        ]
      interval: 60s
      timeout: 5s
      retries: 3
      start_period: 0s
    env_file: .env
    environment:
      DATABASE_CLIENT: ${DATABASE_CLIENT}
      DATABASE_HOST: strapi-db
      DATABASE_PORT: ${DATABASE_PORT}
      DATABASE_NAME: ${DATABASE_NAME}
      DATABASE_USERNAME: ${DATABASE_USERNAME}
      DATABASE_PASSWORD: ${DATABASE_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
      ADMIN_JWT_SECRET: ${ADMIN_JWT_SECRET}
      APP_KEYS: ${APP_KEYS}
    volumes:
      - ./config:/opt/app/config
      - ./src:/opt/app/src
      - ./package.json:/opt/package.json
      - ./public/uploads:/opt/app/public/uploads
    ports:
      - "1337:1337"
    networks:
      - strapi-app-network
    depends_on:
      - strapi-db

  strapi-db:
    container_name: strapi-db
    platform: linux/amd64
    restart: unless-stopped
    env_file: .env
    image: postgres:15.1-alpine
    environment:
      POSTGRES_USER: ${DATABASE_USERNAME}
      POSTGRES_PASSWORD: ${DATABASE_PASSWORD}
      POSTGRES_DB: ${DATABASE_NAME}
    volumes:
      - strapi-app-data:/var/lib/postgresql/data/
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    networks:
      - strapi-app-network

volumes:
  strapi-app-data:

networks:
  strapi-app-network:
    driver: bridge
```

## TypeScript Configuration

To use TypeScript in Strapi, we configure it through a `tsconfig.json` file. This configuration file allows us to specify compiler options and include/exclude files for our project.

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2019",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*.ts", "src/**/*.js"],
  "exclude": ["node_modules", "build", "dist"]
}
```

### `tsconfig.jest.json`

We also have a separate configuration for handling Jest tests.

```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "module": "commonjs"
  },
  "include": ["tests/**/*.ts", "tests/**/*.js"]
}
```

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request with your changes.

## Feedback

If you find this boilerplate useful, don't forget to star the repository ⭐ and share your thoughts through comments! Your feedback helps us improve and grow. ❤️

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
