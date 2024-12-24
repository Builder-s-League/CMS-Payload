<h1 align="center">Payload CMS</h1>

<h3 align="center">
 Reconciliation Road Challenge
</h3>

<br/>

<p align="center">
  <a href="#Requirements"><strong>Requirements</strong></a> ·
  <a href="#Install and Run"><strong>Quick Start</strong></a> ·

</p>
<br/>

# Project Overview
The CMS Payload is where we store content to serve the LMS (Learning Management Service). This CMS acts as a database that provides us with custom structures for the LMS such as topic unit and topic content. 

# Requirements
- Node.js >= 18.17.0
- Docker

# Quick Start

To spin up this example locally, follow these steps:

1. Clone

   If you have not done so already, you need to have a standalone copy of this repo on your machine. If you've already cloned this repo, skip to [Development](#development).
   #### Method 1 (recommended)
   Go to Payload Cloud and [clone this template](https://payloadcms.com/new/clone/website). This will create a new repository on your GitHub account with this template's code which you can then clone to your own machine.

   #### Method 2
   Use the `create-payload-app` CLI to clone this template directly to your machine:

    npx create-payload-app@latest my-project -t websit

   #### Method 3
   Use the `git` CLI to clone this template directly to your machine:

   git clone -n --depth=1 --filter=tree:0 https://github.com/payloadcms/payload my-project && cd my-project && git sparse-checkout set --no-cone templates/website && git checkout && rm -rf .git && git init && git add . && git mv -f templates/website/{.,}* . && git add . && git commit -m "Initial commit"

2. `cp .env.example .env` to copy the example environment variables
3. `yarn` to install dependencies
4. `docker-compose up` to fetch containers and run them (one is a MongoDB container, and one is a container created from the source code of this project)
5. Seed the CMS:

   To seed the database with a few pages, posts, and projects you can run `yarn seed`. This template also comes with a `GET /api/seed` endpoint you can use to seed the database from the admin panel.
   The seed script will also create two users for demonstration purposes only:
   1. Demo Author
      - Email: `demo-author@payloadcms.com`
      - Password: `password`
      - Role: `admin`
   2. Demo User
      - Email: `demo-user@payloadcms.com`
      - Password: `password`
      - Role: `user`

> NOTICE: seeding the database is destructive because it drops your current database to populate a fresh one from the seed template. Only run this command if you are starting a new project or can afford to lose your current data.

# Production

To run Payload in production, you need to build and serve the Admin panel. To do so, follow these steps:

1. Invoke the `payload build` script by running `yarn build` or `npm run build` in your project root. This creates a `./build` directory with a production-ready admin bundle.
1. Finally run `yarn serve` or `npm run serve` to run Node in production and serve Payload from the `./build` directory.
1. When you're ready to go live, see [Deployment](#deployment) for more details.

# How it works

### We only use these collections to develop the CBH LMS app
- #### Topics
  This is where the CMS stores Topics. A Topic can be created here and we can assign Content Items to the Topic we want to. We can understand a Topic as a course unit that has many course contents (Content Items) and we can choose which content we want to be in a created unit.
- #### Content Items
  This is where the CMS stores all of its course content. We can create a new Content Item here using the schema provided by the CMS.

### The below Collections and Globals are just for show and we may remove them in the future

- #### Users (Authentication)

  Users are auth-enabled and encompass both admins and regular users based on the value of their `roles` field. Only `admin` users can access your admin panel to manage your website whereas `user` can authenticate on your front-end to leave [comments](#comments) and read [premium content](#premium-content) but have limited access to the platform. See [Access Control](#access-control) for more details.

  For additional help, see the official [Auth Example](https://github.com/payloadcms/payload/tree/main/examples/auth) or the [Authentication](https://payloadcms.com/docs/authentication/overview#authentication-overview) docs.

- #### Posts

  Posts are used to generated blog posts, news articles, or any other type of content that is published over time. All posts are layout builder enabled so you can generate unique layouts for each post using layout-building blocks, see [Layout Builder](#layout-builder) for more details. Posts are also draft-enabled so you can preview them before publishing them to your website, see [Draft Preview](#draft-preview) for more details.

  Users can also leave comments on posts if they are logged in. Then, editors can log in to review and approve comments before they are published. See [Comments](#comments) for more details.

  Posts can also restrict access to content or digital assets behind authentication, see [Premium Content](#premium-content) for more details.

- ### Comments (Collection)

  Comments are used to allow logged-in users to leave comments on posts. Comments are draft-enabled so admins can review and approve them before they are published to your website, see [Comments](#comments) for more details.

- #### Projects

  Projects are used to showcase your work. All projects are layout builder enabled so you can generate unique layouts for each project using layout-building blocks, see [Layout Builder](#layout-builder) for more details. Projects are also draft-enabled so you can preview them before publishing them to your website, see [Draft Preview](#draft-preview) for more details.

- #### Pages

  All pages are layout builder enabled so you can generate unique layouts for each page using layout-building blocks, see [Layout Builder](#layout-builder) for more details. Pages are also draft-enabled so you can preview them before publishing them to your website, see [Draft Preview](#draft-preview) for more details.

- #### Media

  This is the uploads enabled collection used by pages, posts, and projects to contain media like images, videos, downloads, and other assets.

- #### Categories

  A taxonomy used to group posts or projects together. Categories can be nested inside of one another, for example "News > Technology". See the official [Payload Nested Docs Plugin](https://github.com/payloadcms/plugin-nested-docs) for more details.

### Globals

See the [Globals](https://payloadcms.com/docs/configuration/globals) docs for details on how to extend this functionality.

- `Header`

  The data required by the header on your front-end like nav links.

- `Footer`

  Same as above but for the footer of your site.

## Access control

Basic role-based access control is setup to determine what users can and cannot do based on their roles, which are:

- `admin`: They can access the Payload admin panel to manage your site. They can see all data and make all operations.
- `user`: They cannot access the Payload admin panel and can perform limited operations based on their user (see below).

This applies to each collection in the following ways:

- `users`: Only admins and the user themselves can access their profile. Anyone can create a user but only admins can delete users.
- `posts`: Everyone can access published posts, but only admins can create, update, or delete them. Some posts may also have content that is only accessible to users who are logged in. See [Premium Content](#premium-content) for more details.
- `projects`: Everyone can access published projects, but only admins can create, update, or delete them.
- `pages`: Everyone can access published pages, but only admins can create, update, or delete them.
- `comments`: Everyone can access published comments, but only admins can access draft comments. Users can create new comments but they will be saved as drafts until an admin approves them.

For more details on how to extend this functionality, see the [Payload Access Control](https://payloadcms.com/docs/access-control/overview#access-control) docs.

## Premium Content

Posts can optionally restrict access to content or digital assets behind authentication. This will ensure that only members of your site can access the full post data and its resources. To do this, a `premiumContent` field is added to the `posts` collection with `read` access control set to check for an authenticated user on the request. Every time a user requests a post, this will only return data to those who have access to it:

```ts
{
  name: 'premiumContent',
  label: 'Premium Content',
  type: 'blocks',
  access: {
    read: isLoggedIn,
  },
  fields: [
    // content
  ]
}
```

## Comments

Users can leave comments on posts for editors to review and approve before they are published to the website. To do this, a `comments` collection is added with `drafts` set to `true` so that all comments are saved as drafts and inaccessible until an admin approves them. Each comment references a single `user` and a `doc` for cross reference. To leave a comment you must be logged-in, and to publish a comment you must has the role `admin`.

## Layout Builder

Create unique page, post, or project layouts for any type of content using a powerful layout builder. This template comes pre-configured with the following layout building blocks:

- Hero
- Content
- Media
- Call To Action
- Archive

Each block is fully designed and built into the front-end website that comes with this template. See [Website](#website) for more details.

## Draft Preview

All posts, projects, and pages are draft-enabled so you can preview them before publishing them to your website. To do this, these collections use [Versions](https://payloadcms.com/docs/configuration/collections#versions) with `drafts` set to `true`. This means that when you create a new post, project, or page, it will be saved as a draft and will not be visible on your website until you publish it. This also means that you can preview your draft before publishing it to your website. To do this, we automatically format a custom URL which redirects to your front-end to securely fetch the draft version of your content.

Since the front-end of this template is statically generated, this also means that pages, posts, and projects will need to be regenerated as changes are made to published documents. To do this, we use an `afterChange` hook to regenerate the front-end when a document has changed and its `_status` is `published`.

For more details on how to extend this functionality, see the official [Draft Preview Example](https://github.com/payloadcms/payload/tree/main/examples/draft-preview).

## SEO

This template comes pre-configured with the official [Payload SEO Plugin](https://github.com/payloadcms/plugin-seo) for complete SEO control from the admin panel. All SEO data is fully integrated into the front-end website that comes with this template. See [Website](#website) for more details.

## Redirects

If you are migrating an existing site or moving content to a new URL, you can use the `redirects` collection to create a proper redirect from old URLs to new ones. This will ensure that proper request status codes are returned to search engines and that your users are not left with a broken link. This template comes pre-configured with the official [Payload Redirects Plugin](https://github.com/payloadcms/plugin-redirects) for complete redirect control from the admin panel. All redirects are fully integrated into the front-end website that comes with this template. See [Website](#website) for more details.

## Website

This template includes a beautifully designed, production-ready front-end built with the [Next.js App Router](https://nextjs.org), served right alongside your Payload app in a single Express server. This makes is so that you can deploy both apps simultaneously and host them together. If you prefer a different front-end framework, this pattern works for any framework that supports a custom server. If you prefer to host your website separately from Payload, you can easily [Eject](#eject) the front-end out from this template to swap in your own, or to use it as a standalone CMS. For more details, see the official [Custom Server Example](https://github.com/payloadcms/payload/tree/main/examples/custom-server).

Core features:

- [Next.js App Router](https://nextjs.org)
- [GraphQL](https://graphql.org)
- [TypeScript](https://www.typescriptlang.org)
- [React Hook Form](https://react-hook-form.com)
- [Payload Admin Bar](https://github.com/payloadcms/payload-admin-bar)
- Authentication
- Fully featured blog
- Publication workflow
- Comments
- Premium content
- User accounts
- Dark mode
- Pre-made layout building blocks
- SEO
- Redirects

### Cache

Although Next.js includes a robust set of caching strategies out of the box, Payload Cloud proxies and caches all files through Cloudflare using the [Official Cloud Plugin](https://github.com/payloadcms/plugin-cloud). This means that Next.js caching is not needed and is disabled by default. If you are hosting your app outside of Payload Cloud, you can easily reenable the Next.js caching mechanisms by removing the `no-store` directive from all fetch requests in `./src/app/_api` and then removing all instances of `export const dynamic = 'force-dynamic'` from pages files, such as `./src/app/(pages)/[slug]/page.tsx`. For more details, see the official [Next.js Caching Docs](https://nextjs.org/docs/app/building-your-application/caching).

### Eject

If you prefer another front-end framework or would like to use Payload as a standalone CMS, you can easily eject the front-end from this template. To eject, simply run `yarn eject`. This will uninstall all Next.js related dependencies and delete all files and folders related to the Next.js front-end. It also removes all custom routing from your `server.ts` file and updates your `eslintrc.js`.

> Note: Your eject script may not work as expected if you've made significant modifications to your project. If you run into any issues, compare your project's dependencies and file structure with this template. See [./src/eject](./src/eject) for full details.

For more details on how setup a custom server, see the official [Custom Server Example](https://github.com/payloadcms/payload/tree/main/examples/custom-server).


### Deployment

Before deploying your app, you need to:

1. Ensure your app builds and serves in production. See [Production](#production) for more details.

The easiest way to deploy your project is to use [Payload Cloud](https://payloadcms.com/new/import), a one-click hosting solution to deploy production-ready instances of your Payload apps directly from your GitHub repo. You can also deploy your app manually, check out the [deployment documentation](https://payloadcms.com/docs/production/deployment) for full details.

## Questions

If you have any issues or questions, reach out to us on [Discord](https://discord.com/invite/payload) or start a [GitHub discussion](https://github.com/payloadcms/payload/discussions).
