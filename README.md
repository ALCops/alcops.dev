# alcops.dev

This repository contains the source for [alcops.dev](https://alcops.dev), the home of **ALCops** - A community driven collection of code analyzers for the AL programming language of Microsoft Dynamics 365 Business Central.

The site is built with [Hugo](https://gohugo.io/) using the [Docsy](https://www.docsy.dev/) theme.

## Prerequisites

You need the following tools installed before you can run the site locally:

| Tool | Notes |
|---|---|
| [Hugo Extended](https://gohugo.io/installation/) | The **Extended** edition is required (Docsy uses SCSS). Minimum version: `0.155.0` |
| [Go](https://go.dev/dl/) | Required for Hugo module support |
| [Node.js](https://nodejs.org/) | Required for PostCSS, which Docsy uses for asset processing |

> For full details on Docsy's prerequisites, see the [Docsy getting started guide](https://www.docsy.dev/docs/get-started/docsy-as-module/prerequisites-and-installation/).

## Run locally

1. Clone the repository:

   ```bash
   git clone https://github.com/ALCops/alcops.dev.git
   cd alcops.dev
   ```

2. Install Node.js dependencies:

   ```bash
   npm install
   ```

3. Start the local development server:

   ```bash
   hugo server
   ```

4. Open your browser at `http://localhost:1313`.

Hugo watches for file changes and reloads the browser automatically.

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for how to get started.
