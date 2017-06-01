# Serverless & TypeScript

### 1.6.2017 @ Angular Finland June Meetup

## Jussi Kinnula

---

# Serverless

#### Facts:

- https://serverless.com/
- Run arbitary functions in cloud without setting up a server
- Built originally for AWS Lambda & API Gateway
- Supports also Microsoft, Google & IBM clouds

___

# Serverless

#### Example use:

```
# Install serverless globally
npm install serverless -g

# Create a serverless function
serverless create --template hello-world

# Deploy to cloud provider
serverless deploy

# Function deployed! Trigger with live url
http://xyz.amazonaws.com/hello-world
```

---

# TypeScript

- A typed superset of JavaScript
- Compiles to plain JavaScript
- Used heavily in Angular world, VueJS and Ionic Framework
- From 2.3 supports type-checking errors in `.js` -files with `--checkJs` -option

---

# Setup

- Node Version Manager (nvm)
  + Use multiple NodeJS versions locally
  + For example AWS Lambda runs with NodeJS 6.10
- Serverless
- AWS
  + Command Line Client
  + Defaults
  + IAM credentials

___

# Node Version Manager

Node Version Manager (nvm) enables to use multiple NodeJS versions locally.

```
curl -o- https://raw.githubusercontent.com/ creationix/nvm/v0.33.2/install.sh | bash
```

```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
```

```
nvm install v6.10.3
nvm use v6.10.3
```

___

# Serverless

`npm install -g serverless`

Note! You need this globally to be able to create new Serverless apps (f.e. `sls create --template hello-world`. There can also be local version under project after that.

___

# AWS

#### Command Line Client

`brew install awscli`

```
# aws
usage: aws [options] <command> <subcommand>
  [<subcommand> ...] [parameters]
To see help text, you can run:

  aws help
  aws <command> help
  aws <command> <subcommand> help
aws: error: too few arguments
```

___

# AWS

#### Defaults

Defaults will be used if none defined in `awscli` and `serverless` -commands.

```
# cat ~/.aws/config
[default]
region=eu-west-1
output=json
```

___

# AWS

#### IAM credentials

You need IAM credentials with `AdministratorAccess` role.

```
# cat ~/.aws/credentials
[default]
aws_access_key_id=XXX
aws_secret_access_key=YYY
```

---

## Project structure

- `.webpack/` = build directory for Serverless
- `node_modules/` = installed dependencies
- `package.json` = project configuration
- `serverless.yml` = Serverless configuration
- `src/` = source code
- `target/` = build directory for client and server
- `tsconfig.json` = TypeScript configuration
- `webpack/` = Webpack configuration files
- `webpack.config.ts` = Main Webpack configuration file

___

## Source

- `src/api/users.ts` = Users API endpoint
- `src/client.ts` = Angular application
- `src/index.html` = SPA template
- `src/polyfills.ts` = Polyfills for bundler
- `src/server.ts` = NodeJS & ExpressJS server app
- `src/styles.css` = Styles
- `src/user.model.ts` = User Model

___

### Webpack

- `webpack.config.ts` = builds client and server
- `webpack/webpack.client.ts`
  + The outcome will be in `dist/client.js`, `dist/client.js.map` and `dist/index.html`
- `webpack/webpack.server.ts`
  + The outcome will be in `dist/server.js`
- `webpack/webpack.serverless.ts`
  + The outcome will be in `.webpack/api/users/index.js`

---

# Tooling

___

## Tooling: Client and Server

- `npm install` = install dependencies, and make build if everything was installed successfully
- `npm start` = run production server
- `npm run build` = build client and server
- `npm run build:client` = build client
- `npm run build:server` = build server
- `npm run clean` = cleanup
- `npm run dev` = run webpack-dev-server

___

## Tooling: Serverless

- `npm run sls:clean` = cleanup
- `npm run sls:local users` = run 'users' function locally
- `npm run sls:deploy users` = deploy 'users' function to AWS

---

# Let's code...

---

# Thank you!

GitHub repository (Demo #1 & Demo #2):
https://github.com/jussikinnula/socket-io-and-clouds

GitHub repository (Demo #3):
https://github.com/jussikinnula/ngx-socketio-chat-example

Slides:
https://jussikinnula.github.io/socket-io-and-clouds-wunderdog-tekki-20170512

