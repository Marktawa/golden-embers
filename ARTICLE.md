In this article you will learn how to deploy a [Medusa](https://medusajs.com)-powered ecommerce backend. You will deploy your store's Medusa backend server to [Railway](https://railway.app), the Store Admin frontend to [Cloudflare](https://cloudflare.com)

By the end of this tutorial, you'll have a fully functional Medusa e-commerce backend running in the cloud, ready to serve customers and manage your business. Let's dive in and bring your Medusa store to life on the web!

The method used to deploy Medusa to production is a manual method. In another tutorial I will provide an automated method of deployment to production.

## Prerequisites

Before starting you must have the following:

- [Node.js 16.14.0](https://nodejs.org/en/download/package-manager) or later
- Git - [Download it here](https://git-scm.com/downloads)
- Railway account - [Sign up here](https://railway.app/login)
<!-- Node.js Server running in Railway can be put into sleep mode which further reduces costs. It also means cold starts which can take up to 30 seconds. -->
- GitHub account - [Sign up here](https://github.com/join)
- Cloudflare account - [Sign up here](https://dash.cloudflare.com/sign-up/)
- Neon account - [Sign up here](https://console.neon.tech/signup)

## Create PostgreSQL Database for Medusa on Neon

Using a serverless Postgres database powered by Neon lets you scale down to zero, which helps you save on compute costs.

To get started, go to the [Neon console](https://console.neon.tech/app/projects) and create a project. This creates a Postgres database.

Copy the **Connection String** of your database in the **Connection Details** panel.

![Neon Console Project Dashboard](https://res.cloudinary.com/craigsims808/image/upload/v1722858782/articles/railway-medusa/new-project-neon_iepjio.png)

The Neon connection string will have the following format:

```
postgresql://<user>:<password>@<endpoint_hostname>.neon.tech:<port>/<dbname>?sslmode=require
```

- `<user>` is the database user.
- `<password>` is the database user’s password.
- `<endpoint_hostname>.neon.tech` is the host with neon.tech as the top-level domain (TLD).
- `<port>` is the Neon port number. The default port number is 5432.
- `<dbname>` is the name of the database. neondb is the default database created with each Neon project if you do not define your own.
- `?sslmode=require` is an optional query parameter that enforces SSL mode for better security when connecting to the Postgres instance.

Save the connection string somewhere safe.

## Create a new Medusa commerce application

Medusa is a toolkit for developers to create digital commerce applications. In its simplest form, Medusa is a Node.js backend with the core API, plugins, and modules installed through npm.

<!--
`create-medusa-app` is a command that facilitates creating a Medusa ecosystem. It installs the Medusa backend and admin dashboard, along with the necessary configurations to run the backend.

If you want to connect to a Neon database, you must use the `--db-url` option with its value being the connection string to your Neon database. The connection string you saved in the previous step

In your terminal, run the following command:

```sh
npx create-medusa-app@latest --db-url "postgresql://<user>:<password>@<endpoint_hostname>.neon.tech:<port>/<dbname>?sslmode=require"
```

### Specify project name

After running the command, you will be asked first to enter the name of your project, which is used to create the directory holding your Medusa backend. You can use the default `my-medusa-store` or enter another project name.

### Specify admin email

You'll then be prompted to enter an admin email for your admin user. You'll be using this admin email later to login to your admin dashboard. You can use the default `admin@medusa-test.com` or enter any other email.

After specifying the project name as well as admin email, the Medusa starter backend will be installed.
-->

Make sure your current working directory is the one where you intend to create a project.

In your terminal, run the following command to install the Medusa CLI. We will use it to install the Medusa server.

```bash
npm install @medusajs/medusa-cli -g
```

### Create a new Medusa project

```bash
medusa new my-medusa-store
```

You will be asked to specify your PostgreSQL database credentials. Choose "Skip database setup".

A new directory named `my-medusa-store` will be created to store the server files.

### Configure Database

Add the connection string to your Neon database as the `DATABASE_URL` in your environment variables. Inside `my-medusa-store` create a `.env` file and add the following:

```
DATABASE_URL=postgresql://<user>:<password>@<endpoint_hostname>.neon.tech:<port>/<dbname>?sslmode=require
```

### Seed Database

Run migrations and seed data to the database by running the following command:

```bash
cd my-medusa-store
npx medusa seed --seed-file="./data/seed.json"
```

### Start your Medusa backend

```bash
npx medusa develop
```

The Medusa server will start running on port `9000`.

Test your server:
```bash
curl localhost:9000/store/products
```

If it is working, you should see a list of products.

### Log into Medusa Admin Dashboard

Once the project is prepared, the Medusa backend will start. Visit localhost:7001 to view the admin dashboard in your browser. Use the Email `admin@medusa-test.com` and password `supersecret` to log in.

![Medusa Admin Login](https://res.cloudinary.com/craigsims808/image/upload/v1722860520/articles/railway-medusa/medusa-admin-login_yyrbq8.png)

Once you're logged in, you can start using Medusa!

![Medusa Admin Dashboard](https://res.cloudinary.com/craigsims808/image/upload/v1722860625/articles/railway-medusa/medusa-admin-products_zdplb4.png)

## Deploy Medusa Admin to Cloudflare

We will deploy the Medusa Admin application to Cloudflare Pages.

### Create GitHub Repo

Hosting providers like Cloudflare allow you to deploy your project directly from GitHub. This makes it easier for you to push changes and updates without having to manually trigger the update in the hosting provider.

> **NOTE:**
>
>*Even though you are just deploying the admin, you must include the entire Medusa backend project in the deployed repository. The build process of the admin uses many of the backend's repos.*

<!--
Navigate to your local project directory `medusa-local` of your project folder in your local machine.

Open up `medusa-local` in a new terminal session. Create a new directory named `medusa-admin` inside `medusa-local`.

```sh
mkdir medusa-admin
```

Copy the contents of the `medusa` directory into the newly created `medusa-admin` folder.

```sh
cp -r medusa/ medusa-admin/
```
-->

Copy the contents of your local Medusa project directory, `my-medusa-store` into a new folder. You can name it `medusa-admin`

The `medusa-admin` directory will be used to deploy the Medusa Admin Dashboard.

Create a new GitHub repo based on this directory to handle all the config related to the admin only.

### Configure Build Command

In the `package.json` file of `medusa-admin`, change the build script for the admin as follows:

```json
"scripts": {
  //other scripts
  "build:admin": "medusa-admin build --deployment",
}
```

> **NOTE:**
>
> *When using `--deployment` option, the backend's URL is loaded from the `MEDUSA_ADMIN_BACKEND_URL` environment variable. You will configure this environment variable in a later step.*

### Preparing Deployment

Log in to your [Cloudflare Dashboard](https://dash.cloudflare.com/) and select **Workers & Pages**.

![Cloudflare Dashboard](https://res.cloudinary.com/craigsims808/image/upload/v1720439036/articles/medusa-sveltekit/cloudflare-dashboard_q2rvwf.png)

Select **Create application** then **Pages** then **Connect to Git**.

You will be prompted to sign in with your preferred Git provider.

Next, select the GitHub project for your Medusa Admin repo.

Once you have selected a repository, select **Install & Authorize** and **Begin setup**. 

![Select GitHub repo](https://res.cloudinary.com/craigsims808/image/upload/v1720439067/articles/medusa-sveltekit/select-github-repo-cloudflare_ea42tu.png)

You can then customize your deployment in **Set up builds and deployments**.

Your **project name** will be used to generate your project's hostname.

### Configure Build settings

Set the build command of your deployed project to use the `build:admin` command:

```bash
npm run build:admin
```

Set the output directory of your deployed project to `build`.

Add the environment variable `MEDUSA_ADMIN_BACKEND_URL` and set its value to `localhost:9000` provisionally.

![Cloudflare Set up builds and deployments](https://res.cloudinary.com/craigsims808/image/upload/v1723981792/articles/railway-medusa/admin-build-settings_flbrit.png)

### Configure paths

To configure which paths are included and excluded, go to your Pages project > Settings > Builds & deployments > Build watch paths.

Trigger the build when changes occur within the `medusa-admin` directory:

- Include paths: `medusa-admin`
- Exclude paths: ``

### Save Configuration and Deploy Admin

After you have finished setting your build configuration, select **Save and Deploy**. Your project build logs will output as Cloudflare Pages installs your project dependencies, builds the project, and deploys it to Cloudflare’s global network.

When your project has finished deploying, you will receive a unique URL to view your deployed site. This unique URL will be used as the value for the `ADMIN_CORS` environment variable when you deploy the Medusa backend server.

![Cloudflare Page Admin URL](https://res.cloudinary.com/craigsims808/image/upload/v1720439037/articles/medusa-sveltekit/cloudflare-admin-deploy-success_sy3uum.png)

## Deploy Medusa Server to Railway

We will deploy the Medusa Backend Server on [Railway](https://railway.app/). Railway provides a free trial (no credit card required) that allows you to deploy your Medusa backend along with PostgreSQL and Redis databases. This is useful mainly for development and demo purposes. [Sign up for a Railway account](https://railway.app/login) if you haven't already and proceed with the following steps.

### Create GitHub repo

<!--
Navigate to your local Medusa store directory `medusa-local`. Create a new GitHub repo using the `medusa-backend-prod` directory. This will be the GitHub repo that Railway will use for deploying the Medusa backend server for your store.
-->

Create a new directory named `medusa-server`.

```sh
mkdir medusa-server
```

Copy the contents of your local Medusa project, `my-medusa-store` directory into the newly created `medusa-server` folder.

```sh
cp -r my-medusa-store/ medusa-server/
```

Create a new GitHub repo using the `medusa-server` directory. This will be the GitHub repo that Railway will use for deploying the Medusa backend server for your store.

### Add Railway Configuration File

Add in the root of `medusa-server` the file, `railway.toml`, with the content based on the package manager of your choice:

> **NOTE**:
>
> *To avoid errors during the installation process, it's recommended to use `yarn` for installing the dependencies. Alternatively, pass the `--legacy-peer-deps` option to the `npm` command.*

Using `yarn`:

```toml
[build]
builder = "NIXPACKS"

[build.nixpacksPlan.phases.setup]
nixPkgs = ["nodejs", "yarn"]

[build.nixpacksPlan.phases.install]
cmds=["yarn install"]
```

Or using `npm`:

```toml
[build]
builder = "NIXPACKS"

[build.nixpacksPlan.phases.setup]
nixPkgs = ["nodejs", "npm"]

[build.nixpacksPlan.phases.install]
cmds=["npm install --legacy-peer-deps"]
```

### Configure Server for Production

Open the `medusa-config.js` file in your `medusa-server` directory.

Update the following parts to enable caching using Redis.

Uncomment the inner part of the following section:

```js
const modules = {
  /*eventBus:
    resolve: "@medusajs/event-bus-redis",
```

Uncomment the following section as well:

```js
// Uncomment the following lines to enable REDIS
// redis_url: REDIS_URL
```

It then becomes:

```js
//...
const modules = {
  eventBus: {
    resolve: "@medusajs/event-bus-redis",
    options: {
      redisUrl: REDIS_URL
    }
  },
  cacheService: {
    resolve: "@medusajs/cache-redis",
    options: {
      redisUrl: REDIS_URL
    }
  },
};

/** @type {import('@medusajs/medusa').ConfigModule["projectConfig"]} */
const projectConfig = {
  jwtSecret: process.env.JWT_SECRET,
  cookieSecret: process.env.COOKIE_SECRET,
  store_cors: STORE_CORS,
  database_url: DATABASE_URL,
  admin_cors: ADMIN_CORS,
  redis_url: REDIS_URL
};
//...
```

Commit your changes, and push them to your remote GitHub repository. Once your repository is ready on GitHub, log in to your Railway dashboard.

### Create Project + Redis

On the Railway Dashboard, click on the **New Project** button and choose from the list the **Deploy Redis** option.

![New Railway Project](https://res.cloudinary.com/craigsims808/image/upload/v1723985040/articles/railway-medusa/new-railway-project-redis_cs0loe.png)

A Redis instance will be created and, after a few seconds, you will be redirected to your project page.

![Railway Project Dashboard](https://res.cloudinary.com/craigsims808/image/upload/v1723985261/articles/railway-medusa/project-dashboard-railway_odaw0i.png)

### Deploy Medusa in Server Mode

In this section, you'll create a Medusa backend instance running in `server` runtime mode.

In the same project view, click on the **Create** button and choose the **GitHub Repo** option. If you still haven't given GitHub permissions to Railway, choose the **Configure GitHub App** option.

![Configure GitHub Repo on Railway](https://res.cloudinary.com/craigsims808/image/upload/v1720439054/articles/medusa-sveltekit/railway-add-github-repo_cetjbf.png)

### Configure Medusa Backend Environment Variables

To configure the environment variables of your Medusa backend, click on the GitHub repo card and choose the **Variables** tab and add the following environment variables:

```
PORT=9000
JWT_SECRET=something
COOKIE_SECRET=something
DATABASE_URL=postgresql://dominggobana:JyyuEdr809p@df-hidden-bonus-ertd7sio.us-east-3.aws.neon.tech/storedb?sslmode=require
REDIS_URL=${{Redis.REDIS_URL}}
ADMIN_CORS=<CLOUDFLARE-ADMIN-URL>
```

- `JWT_SECRET` and `COOKIE_SECRET` should be unique and secure values
- `DATABASE_URL` is the connection string to your Neon hosted Postgres database
- `REDIS_URL` references the value from the Redis database you created earlier.
`ADMIN_CORS` should be replaced with the URL generated from the admin deployed on Cloudflare.

### Configure Start Command

Click on the GitHub repository's card, select the **Settings** tab and scroll down to the **Deploy** section. Click on the **Start** command button and paste the following command:

```sh
medusa migrations run && medusa start
```

![Change Start command in Railway](https://res.cloudinary.com/craigsims808/image/upload/v1720439054/articles/medusa-sveltekit/railway-change-start_zqbgyr.png)

### Define Root Directory for Deployment

Select the **Settings** tab and scroll down to the **Root Directory** section. Set the directory to the `medusa-server` directory containing your Medusa app code.

### Set Watch paths

To prevent unnecessary rebuilds, it is important to set the watch paths. Set the watch-paths to `medusa-server/**`.

### Add Domain Name

Click on the Medusa server runtime card and select the **Settings** tab and scroll down to the **Networking** section. Either select **Custom Domain** or select **Generate Domain** to generate a random button.

![Add Domain Name to Railway Project](/railway-generate-domain.png)

### Deploy Changes

At the top left of your project's dashboard page, there's a **Deploy** button that deploys all the changes you've made so far.

![Railway Deploy button](https://res.cloudinary.com/craigsims808/image/upload/v1720439056/articles/medusa-sveltekit/railway-deploy-button_keobv6.png)

Click on the button to trigger the deployment. The deployment will take a few minutes before it's ready.

### Test the Backend

Once the deployment is finished, you can access the Medusa backend on the custom domain/domain you've generated.

For example, you can open the URL `<YOUR_APP_URL>/store/products` which returns the products available on your backend.

![Railway Deployment Test](https://res.cloudinary.com/craigsims808/image/upload/v1720439059/articles/medusa-sveltekit/railway-deploy-test_hempr1.png)

### Health Route

Access `<YOUR_APP_URL>/health` to get health status of your deployed backend.

![Raliway Deployment Health](https://res.cloudinary.com/craigsims808/image/upload/v1720439056/articles/medusa-sveltekit/railway-deploy-health_gv1vgs.png)

### Test the Admin

Revisit your Cloudflare Dashboard and open the **Settings** page for your live admin project. Update the value for environment variable `MEDUSA_ADMIN_BACKEND_URL` with the URL to your deployed Medusa backend server on Railway.

Save the settings and rebuild your admin deployment.

Visit the URL to your Medusa Admin and log in using the user you created in the previous steps.

![Medusa Admin on Cloudflare Pages](https://res.cloudinary.com/craigsims808/image/upload/v1720439036/articles/medusa-sveltekit/cloudflare-admin-login_xvkl9u.png)

If all is working you should be able to log in to your Admin dashboard.

![Medusa Admin Dashboard Logged In](https://res.cloudinary.com/craigsims808/image/upload/v1720439052/articles/medusa-sveltekit/medusa-admin-dashboard-loggedin-cloudflare_gxnlze.png)

## Conclusion

Congratulations! You've successfully deployed your Medusa e-commerce backend to production. Your backend is now running on Railway, while your admin dashboard is hosted on Cloudflare Pages. This setup provides a robust, scalable foundation for your online business.

By following this guide, you've learned how to:

- Set up the Medusa admin dashboard on Cloudflare Pages
- Configure and deploy the Medusa backend server on Railway
- Connect all components to work seamlessly together

With your store now live, you're ready to start customizing your shop, adding products, and processing orders. Remember to regularly update your deployment as new features and security patches are released for Medusa.
As you continue to grow your e-commerce business, explore Medusa's extensive plugin ecosystem and advanced features to enhance your store's functionality. Happy selling!

## Resources

[GitHub Repo](https://github.com/Marktawa/golden-embers)

## Author

[Mark Munyaka](https://marktawa.github.io/personal-portfolio)

GitHub: [@Marktawa](https://github.com/Marktawa)  
Twitter: [@McMunyaka](https://twitter.com/McMunyaka)

## Hire Me

If you found this guide helpful and would like assistance in setting up your own Medusa-powered ecommerce store, I would be happy to help!
As an experienced Medusa and Next.js developer, I can provide the following services:

- Initial setup and configuration of your Medusa backend and Next.js storefront
- Deployment of your Medusa store on AWS, Railway, Digital Ocean and any other platform
- Integration with Neon Postgres, Stripe, Meilisearch, SendGrid, and other plugins
- Customization and branding of your store's design and functionality
- Ongoing maintenance, updates, and feature enhancements
- Performance optimization and scaling guidance

To discuss your project and get a quote, please feel free to contact me on [markmunyakapro@gmail.com](mailto:markmunyakapro@gmail.com) or connect with me on [LinkedIn](https://www.linkedin.com/in/mark-tawanda-munyaka-a878137b) or [Discord](https://discord.com/users/558354133170782254). I look forward to hearing from you!

## Sponsor

Support my passion for sharing development knowledge by making a donation to my [**Buy Me a Coffee**](https://www.buymeacoffee.com/markmunyaka) account. Your contribution helps me create valuable content and resources. Thank you for your support!

[![Buy Me A Coffee Banner](https://res.cloudinary.com/craigsims808/image/upload/v1708089939/articles/test/buymeacoffee_lqmwjn.png)](https://www.buymeacoffee.com/markmunyaka)

[Buy Me A Coffee](https://www.buymeacoffee.com/markmunyaka)
