<h2 align="center">gatsby-v5-source-hygraph</h2>
<p align="center">A Gatsby v5 source plugin for Hygraph projects</p>

<p align="center">
  <a href="https://npmjs.org/package/gatsby-v5-source-hygraph">
    <img src="https://img.shields.io/npm/v/gatsby-v5-source-hygraph.svg" alt="Version" />
  </a>
  <a href="https://npmjs.org/package/gatsby-v5-source-hygraph">
    <img src="https://img.shields.io/npm/dw/gatsby-v5-source-hygraph.svg" alt="Downloads/week" />
  </a>
  <a href="https://github.com/jonvisc/gatsby-v5-source-hygraph/blob/main/LICENSE">
    <img src="https://img.shields.io/npm/l/gatsby-v5-source-hygraph.svg" alt="License" />
  </a>
  <a href="https://github.com/jonvisc/gatsby-v5-source-hygraph/stargazers">
    <img src="https://img.shields.io/github/stars/hygraph/gatsby-v5-source-hygraph" alt="Forks on GitHub" />
  </a>
  <img src="https://badgen.net/bundlephobia/minzip/gatsby-v5-source-hygraph" alt="minified + gzip size" />
  <!-- ALL-CONTRIBUTORS-BADGE:START - Do not remove or modify this section -->
<img src="https://img.shields.io/badge/all_contributors-1-purple.svg" alt="Contributors" />
<!-- ALL-CONTRIBUTORS-BADGE:END -->
</p>

## Installation

```shell
yarn add gatsby-v5-source-hygraph gatsby-plugin-image
```

> Tested on Gatsby v5, Node 20.11.0

## Configuration

> We recommend using environment variables with your Hygraph `token` and `endpoint` values. You can learn more about using environment variables with Gatsby [here](https://www.gatsbyjs.org/docs/environment-variables).

### Basic

```js
// gatsby-config.js
module.exports = {
  plugins: [
    {
      resolve: 'gatsby-v5-source-hygraph',
      options: {
        endpoint: process.env.HYGRAPH_ENDPOINT,
      },
    },
  ],
}
```

### Authorization

You can also provide an auth token using the `token` configuration key. This is necessary if your Hygraph project is **not** publicly available, or you want to scope access to a specific content stage (i.e. draft content).

```js
// gatsby-config.js
module.exports = {
  plugins: [
    {
      resolve: 'gatsby-v5-source-hygraph',
      options: {
        endpoint: process.env.HYGRAPH_ENDPOINT,
        token: process.env.HYGRAPH_TOKEN,
      },
    },
  ],
}
```

### Options

| Key                   | Type                                     | Description                                                                                                                                                                                                                                                                                                                         |
| --------------------- | ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `endpoint`            | String (**required**)                    | The endpoint URL for the Hygraph project. This can be found in the [project settings UI](https://hygraph.com/docs/guides/concepts/apis#working-with-apis).                                                                                                                                                                          |
| `token`               | String                                   | If your Hygraph project is **not** publicly accessible, you will need to provide a [Permanent Auth Token](https://hygraph.com/docs/reference/authorization) to correctly authorize with the API. You can learn more about creating and managing API tokens [here](https://hygraph.com/docs/guides/concepts/apis#working-with-apis). |
| `typePrefix`          | String _(Default: `Hygraph_`)\_         | The string by which every generated type name is prefixed with. For example, a type of `Post` in Hygraph would become `Hygraph_Post` by default. If using multiple instances of the source plugin, you **must** provide a value here to prevent type conflicts.                                                                    |
| `downloadLocalImages` | Boolean _(Default: `false`)_             | Download and cache Hygraph image assets in your Gatsby project. [Learn more](#downloading-local-image-assets).                                                                                                                                                                                                                      |
| `buildMarkdownNodes`  | Boolean _(Default: `false`)_             | Build markdown nodes for all [`RichText`](https://hygraph.com/docs/reference/fields/rich-text) fields in your Hygraph schema. [Learn more](#using-markdown-nodes).                                                                                                                                                                  |
| `fragmentsPath`       | String _(Default: `hygraph-fragments`)_ | The local project path where generated query fragments are saved. This is relative to your current working directory. If using multiple instances of the source plugin, you **must** provide a value here to prevent type and/or fragment conflicts.                                                                                |
| `locales`             | String _(Default: `['en']`)_             | An array of locale key strings from your Hygraph project. [Learn more](#querying-localised-nodes). You can read more about working with localisation in Hygraph [here](https://hygraph.com/docs/guides/concepts/i18n).                                                                                                              |
| `stages`              | String _(Default: `['PUBLISHED']`)_      | An array of Content Stages from your Hygraph project. [Learn more](#querying-from-content-stages). You can read more about using Content Stages [here](https://hygraph.com/guides/working-with-content-stages).                                                                                                                     |
| `queryConcurrency`    | Integer _(Default: 10)_                  | The number of promises ran at once when executing queries.                                                                                                                                                                                                                                                                          |

## Features

- [Installation](#installation)
- [Configuration](#configuration)
  - [Basic](#basic)
  - [Authorization](#authorization)
  - [Options](#options)
- [Features](#features)
  - [Querying localised nodes](#querying-localised-nodes)
  - [Querying from content stages](#querying-from-content-stages)
  - [Usage with `gatsby-plugin-image`](#usage-with-gatsby-plugin-image)
    - [`gatsbyImageData` resolver arguments](#gatsbyimagedata-resolver-arguments)
  - [Downloading local image assets](#downloading-local-image-assets)
  - [Using markdown nodes](#using-markdown-nodes)
    - [Usage with `gatsby-plugin-mdx`](#usage-with-gatsby-plugin-mdx)
  - [Working with query fragments](#working-with-query-fragments)
    - [Modifying query fragments](#modifying-query-fragments)
- [Contributors](#contributors)
- [FAQs](#faqs)

### Querying localised nodes

If using Hygraph localisation, this plugin provides support to build nodes for all provided locales.

Update your plugin configuration to include the `locales` key.

```js
// gatsby-config.js
module.exports = {
  plugins: [
    {
      resolve: 'gatsby-v5-source-hygraph',
      options: {
        endpoint: process.env.HYGRAPH_ENDPOINT,
        locales: ['en', 'de'],
      },
    },
  ],
}
```

To query for nodes for a specific locale, use the `filter` query argument.

```gql
{
  enProducts: allHygraphProduct(filter: { locale: { eq: en } }) {
    nodes {
      name
    }
  }
}
```

### Querying from content stages

This plugin provides support to build nodes for entries from multiple Content Stages.

The provided Content Stages **must** be accessible according to the configuration of your project's [API access](https://hygraph.com/docs/authorization). If providing a `token`, then that [Permanent Auth Token](https://hygraph.com/docs/authorization#permanent-auth-tokens) must have permission to query data from all provided Content Stages.

The example below assumes that both the `DRAFT` and `PUBLISHED` stages are publicly accessible.

```js
// gatsby-config.js
module.exports = {
  plugins: [
    {
      resolve: 'gatsby-v5-source-hygraph',
      options: {
        endpoint: process.env.HYGRAPH_ENDPOINT,
        stages: ['DRAFT', 'PUBLISHED'],
      },
    },
  ],
}
```

To query for nodes from a specific Content Stage, use the `filter` query argument.

```gql
{
  allHygraphProduct(filter: { stage: { eq: DRAFT } }) {
    nodes {
      name
    }
  }
}
```

### Usage with `gatsby-plugin-image`

> Requires [`gatsby-plugin-image`](https://www.gatsbyjs.com/plugins/gatsby-plugin-image) as a project dependency.

This source plugin supports `gatsby-plugin-image` for responsive, high performance Hygraph images direct from our CDN.

Use the `gatsbyImageData` resolver on your `Hygraph_Asset` nodes.

```gql
{
  allHygraphAsset {
    nodes {
      gatsbyImageData(layout: FULL_WIDTH)
    }
  }
}
```

#### `gatsbyImageData` resolver arguments

| Key                    | Type                                                   | Description                                                                                                                                                                                                                                                                                                                                                                                                   |
| ---------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `aspectRatio`          | Float                                                  | Force a specific ratio between the image’s width and height.                                                                                                                                                                                                                                                                                                                                                  |
| `backgroundColor`      | String                                                 | Background color applied to the wrapper.                                                                                                                                                                                                                                                                                                                                                                      |
| `breakpoints`          | [Int]                                                  | Output widths to generate for full width images. Default is to generate widths for common device resolutions. It will never generate an image larger than the source image. The browser will automatically choose the most appropriate.                                                                                                                                                                       |
| `height`               | Int                                                    | Change the size of the image.                                                                                                                                                                                                                                                                                                                                                                                 |
| `layout`               | GatsbyImageLayout (`CONSTRAINED`/`FIXED`/`FULL_WIDTH`) | Determines the size of the image and its resizing behavior.                                                                                                                                                                                                                                                                                                                                                   |
| `outputPixelDensities` | [Float]                                                | A list of image pixel densities to generate. It will never generate images larger than the source, and will always include a 1x image. The value is multiplied by the image width, to give the generated sizes. For example, a `400` px wide constrained image would generate `100`, `200`, `400` and `800` px wide images by default. Ignored for full width layout images, which use `breakpoints` instead. |
| `quality`              | Int                                                    | The default image quality generated. This is overridden by any format-specific options.                                                                                                                                                                                                                                                                                                                       |
| `sizes`                | String                                                 | [The `<img> sizes` attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#attributes), passed to the img tag. This describes the display size of the image, and does not affect generated images. You are only likely to need to change this if your are using full width images that do not span the full width of the screen.                                                             |
| `width`                | Int                                                    | Change the size of the image.                                                                                                                                                                                                                                                                                                                                                                                 |
| `placeholder`          | `NONE`/`BLURRED`/`DOMINANT_COLOR`/`TRACED_SVG`         | Choose the style of temporary image shown while the full image loads.                                                                                                                                                                                                                                                                                                                                         |

**NOTE**: `gatsby-plugin-sharp` needs to be listed as a dependency on your project if you plan to use placeholder `TRACED_SVG` or `DOMINANT_COLOR`.

For more information on using `gatsby-plugin-image`, please see the [documentation](https://www.gatsbyjs.com/plugins/gatsby-plugin-image/).

### Downloading local image assets

If you prefer, the source plugin also provides the option to download and cache Hygraph assets in your Gatsby project.

To enable this, add `downloadLocalImages: true` to your plugin configuration.

```js
// gatsby-config.js
module.exports = {
  plugins: [
    {
      resolve: 'gatsby-v5-source-hygraph',
      options: {
        endpoint: process.env.HYGRAPH_ENDPOINT,
        downloadLocalImages: true,
      },
    },
  ],
}
```

This adds a `localFile` field to the `Hygraph_Asset` type which resolves to the file node generated at build by [`gatsby-source-filesystem`](https://www.gatsbyjs.org/packages/gatsby-source-filesystem).

```gql
{
  allHygraphAsset {
    nodes {
      localFile {
        childImageSharp {
          gatsbyImageData(layout: FULL_WIDTH)
        }
      }
    }
  }
}
```

### Using markdown nodes

This source plugin provides the option to build markdown nodes for all `RichText` fields in your Hygraph schema, which in turn can be used with [MDX](https://mdxjs.com).

To enable this, add `buildMarkdownNodes: true` to your plugin configuration.

```js
// gatsby-config.js
module.exports = {
  plugins: [
    {
      resolve: 'gatsby-v5-source-hygraph',
      options: {
        endpoint: process.env.HYGRAPH_ENDPOINT,
        buildMarkdownNodes: true,
      },
    },
  ],
}
```

Enabling this option adds a `markdownNode` nested field to all `RichText` fields on the generated Gatsby schema.

You will need to rebuild your `hygraph-fragments` if you enable embeds on a Rich Text field, or you add/remove additional fields to your Hygraph schema.

#### Usage with `gatsby-plugin-mdx`

These newly built nodes can be used with [`gatsby-plugin-mdx`](https://www.gatsbyjs.org/packages/gatsby-plugin-mdx) to render markdown from Hygraph.

Once installed, you will be able to query for `MDX` fields using a query similar to the one below.

```gql
{
  allHygraphPost {
    nodes {
      id
      content {
        markdownNode {
          childMdx {
            body
          }
        }
      }
    }
  }
}
```

### Working with query fragments

The source plugin will generate and save GraphQL query fragments for every node type. By default, they will be saved in a `hygraph-fragments` directory at the root of your Gatsby project. This can be configured:

> If using multiple instances of the source plugin, you **must** provide a value to prevent type and/or fragment conflicts.

```js
// gatsby-config.js
module.exports = {
  plugins: [
    {
      resolve: 'gatsby-v5-source-hygraph',
      options: {
        endpoint: process.env.HYGRAPH_ENDPOINT,
        fragmentsPath: 'my-query-fragments',
      },
    },
  ],
}
```

The generated fragments are then read from the project for subsequent builds. It is recommended that they are checked in to version control for your project.

Should you make any changes or additions to your Hygraph schema, you will need to update the query fragments accrdingly. Alternatively they will be regnerated on a subsequent build after removing the directory from your project.

#### Modifying query fragments

In some instances, you may need modify query fragments on a per type basis. This may involve:

- Removing unrequired fields
- Adding new fields with arguments as an aliased field

For example, adding a `featuredCaseStudy` field:

```graphql
fragment Industry on Industry {
  featuredCaseStudy: caseStudies(where: { featured: true }, first: 1)
}
```

Field arguments cannot be read by Gatsby from the Hygraph schema. Instead we must alias any required usages as aliased fields. In this example, the `featuredCaseStudy` field would then be available in our Gatsby queries:

```graphql
{
  allHygraphIndustry {
    nodes {
      featuredCaseStudy {
        ...
      }
    }
  }
}
```

If you spot an issue, or bug that you know how to fix, please submit a pull request.

When working locally you'll need to run `yarn compile` and `yarn dev` using Yarn workspaces that will let you test/develop on the `demo` Gatsby app in this project. You may need to `yarn clean` occcasionally too.

## FAQs

<details>
  <summary>"endpoint" is required</summary>

If you are using environment variables, make sure to include `require("dotenv").config();` inside your `gatsby-config.js`.

If it's already included, make sure you have your ENV variable added to `.env`, or `.env.local` without spaces.

</details>

<details>
  <summary>"message": "not allowed"</summary>

This error occurs most likely if your token doesn't have access to the `PUBLISHED` content stage. Configure your token to also access `PUBLISHED`, or specify `stages: ["DRAFT"]` to the options inside `gatsby-config.js`.

</details>

<details>
  <summary>Bad request</summary>

You may need to rebuild your fragments folder when making schema changes. If you change the type of a field, or add/remove any from an existing model you have fragments for, the plugin cannot query for this.

Simply delete the `hygraph-fragments` (or whatever you named it), and run `gatsby develop` to regenerate.

</details>
