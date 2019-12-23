# Stremio Addon SDK 🧙

![Stremio](https://www.stremio.com/website/stremio-purple-small.png)

The **🧙  Stremio Addon SDK 🧙** was developed by the Stremio Team as a way of vastly simplifying Node.js addon creation for
our streaming platform.

Stremio currently supports Windows, macOS, Linux, Android and iOS.


## Quick Example

This arbitrary example creates an addon that provides a stream for Big Buck Bunny and outputs a HTTP address where you can access it.

```javascript
const { addonBuilder, serveHTTP, publishToCentral }  = require('stremio-addon-sdk')

const builder = new addonBuilder({
    id: 'org.myexampleaddon',
    version: '1.0.0',

    name: 'simple example',

    // Properties that determine when Stremio picks this addon
    // this means your addon will be used for streams of the type movie
    catalogs: [],
    resources: ['stream'],
    types: ['movie'],
    idPrefixes: ['tt']
})

// takes function(args)
builder.defineStreamHandler(function(args) {
    if (args.type === 'movie' && args.id === 'tt1254207') {
        // serve one stream to big buck bunny
        const stream = { url: 'http://distribution.bbb3d.renderfarming.net/video/mp4/bbb_sunflower_1080p_30fps_normal.mp4' }
        return Promise.resolve({ streams: [stream] })
    } else {
        // otherwise return no streams
        return Promise.resolve({ streams: [] })
    }
})

serveHTTP(builder.getInterface(), { port: 7000 })
//publishToCentral("https://your-domain/manifest.json") // <- invoke this if you want to publish your addon and it's accessible publically on "your-domain"
```

Save this as `addon.js` and run:

```bash
npm install stremio-addon-sdk
node ./addon.js
```

It will output a URL that you can use to [install the addon in Stremio](./docs/testing.md#how-to-install-add-on-in-stremio)

**Please note:** addon URLs in Stremio must be loaded with HTTPS (except `127.0.0.1`) and must support CORS! CORS support is handled automatically by the SDK, but if you're trying to load your addon remotely (not from `127.0.0.1`), you need to support HTTPS.


## Getting started with a new addon

In order to scaffold a new Stremio addon, we've made a tool called `addon-bootstrap`.

You can use it in the following way:

```bash
npm install -g stremio-addon-sdk # use sudo if on Linux
addon-bootstrap hello-world
```

You'll be asked about what [resources and types](./docs/api/README.md) you want to support, after which the addon will be created in the `hello-world` directory, and you'll be able to run it:

```bash
cd hello-world
npm install
npm start -- --launch
```

If you wish to install the addon in the Desktop version of Stremio (which you can [download here](https://www.stremio.com/downloads)), you should use `npm start -- --install`

## Documentation

All our documentation is [right here on GitHub](./docs). Take a look at our [examples list](./docs/examples.md) for some high-level
information, or dive straight into our [SDK documentation](./docs/README.md) for our code reference docs.

We also have an [example addon](https://github.com/Stremio/addon-helloworld) that you can use as a guide to help you build your own addon.

We've made two step by step guides: one for this SDK, and one for any programming language, [which you can read here](https://stremio.github.io/stremio-addon-guide).

If you don't wish to use Node.js (and therefore not use this SDK either), you can create addons in any programming
language, see the [addon protocol specification](./docs/protocol.md) for more information.

It is also possible to create an addon without any programming language, see our [static addon example](https://github.com/Stremio/stremio-static-addon-example) based
on the protocol specification.

SDK Features Include:

- Publishing an addon through HTTP(s)
- Publishing an addon through IPFS
- Publishing your addon link to the [public Addon collection](https://api.strem.io/addonscollection.json) with [publishToCentral](./docs/README.md#publishtocentralurl)
- Creating a homepage for your addon that includes an "Install Addon" button

## Testing

For developers looking for a quick way to test their new addons, you can either:

- [Test with Stremio](./docs/testing.md#testing-in-stremio-app)
- [Test with our Web Version](./docs/testing.md#testing-in-stremio-web-version)


## Deploying

In order for your addon to be used by others, it needs to be deployed online.

You can check our [list of recommended hosting providers for Node.js](./docs/deploying/README.md) or alternatively host it locally with [localtunnel](https://github.com/localtunnel/localtunnel).

After you've deployed publically, in order to get your addon to show in Stremio (through the [public Addon collection](https://api.strem.io/addonscollection.json)), you need to use [publishToCentral](./docs/README.md#publishtocentralurl) or publish [manually through the UI](https://stremio.github.io/stremio-publish-addon/index.html).


### IPFS deployment

You can optionally deploy your addon to IPFS, which is very similar to a P2P Torrent network. It is recommended to publish to IPFS to ensure addon longevity and your privacy.

In order to use the IPFS features of `stremio-addon-sdk`, you will need to also:

- use a node.js version of at least v11.10.0

- `npm install stremio-addon-ipfs` this module is required because it includes all IPFS specific dependencies

- IPFS installed locally, see [this page](https://docs.ipfs.io/guides/guides/install/) for details


#### IPFS Publish

Publishing to IPFS requires a Supernode, if one is not given, then a random remote supernode will be selected automatically.

`./cli/publish.js <addonUrl>` - publish the addon at the provided transport URL

Options:

- `--help`: Show help
- `--version`: Show version number
- `--supernode`: Address of the supernode (defaults to a random remote supernode), if you are running the supernode locally, then use: "ws://127.0.0.1:14011"
- `--peer-id`: Peer ID of the supernode (typically found at `~/.ipfs/config`), setting this bypasses the requirement to port forward inbound port 4001
- `--restoreFromMnemonic`: Restore publishing identity from BIP39 mnemonic


#### IPFS Supernode

Supernodes will act as the seeds to your addon, handling it's requests, caching the data and serving it to others too. You can run a Supernode locally, deploy it to a remote server or use one that is already deployed somewhere.

`./cli/supernode.js` - runs a server that facilitates serving addons from IPFS

Environment variables:

- `PORT`: port to listen on (only applies to supernode)

- `IPFS_MULTIADDR`: Multiaddr of the IPFS daemon; default value is `/ip4/127.0.0.1/tcp/5001`


## Examples & tutorials

Check out our ever growing list of [examples and demo addons](./docs/examples.md). This list also includes examples & tutorials on how to develop Stremio addons in PHP, Python, Ruby, C#, Java and Go. It also includes a list of video tutorials.


## Advanced Usage

Read our [guide for advanced usage](./docs/advanced.md) to understand the many ways that addons can be used.


## Reporting Issues

If you have any issues regarding the Stremio Addon SDK, please feel free to [report them here](https://github.com/Stremio/stremio-addon-sdk/issues).


## Migration from v0.x

To migrate from v0.x, you need to:

- change `new addonSDK` to `new addonBuilder`, which you can import via `const addonBuilder = require('stremio-addon-sdk').addonBuilder`
- change `addon.run(opts)` to `serveHTTP(addon.getInterface(), opts)`, which you can import via `const serveHTTP = require('stremio-addon-sdk').serveHTTP`
- all handlers have to return a `Promise` (rather than take a `cb`)


## Use Cases Outside Addon SDK

The use of this SDK is not mandatory for creating Stremio Addons. You can use any programming language that supports
creating a HTTP server to make Stremio Addons. Refer to our [protocol specification](./docs/protocol.md) for details and examples.

One useful scenario of not using the SDK is when you need user specific data for you addon (for example, an API
Autherntication Token), you can see an example of passing user specific data in the Addon URL [here](./docs/advanced.md#using-user-data-in-add-ons).
This example uses Node.js and Express to get user specific data.


_built with love and serious coding skills by the Stremio Team_

<img src="https://blog.stremio.com/wp-content/uploads/2018/03/new-logo-cat-blog.jpg" width="300" />
