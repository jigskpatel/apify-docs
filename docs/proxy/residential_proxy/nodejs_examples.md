---
title: NodeJS Examples
description: Learn how to connect to Apify's residential proxies from your application using Node.js code examples. Configure proxy locations and IP address use.
paths:
    - proxy/residential-proxy/nodejs-examples
---

# [](#nodejs-examples)NodeJS Examples

The following sections contain several examples of how to use Apify Proxy in NodeJS (used as the default language in [actors]({{@link actors.md}})).

## [](#usage-in-puppeteer-crawler) Usage in [PuppeteerCrawler](https://sdk.apify.com/docs/api/puppeteer-crawler)

Use a single session with IP from the US for the whole PuppeteerCrawler run (for as long as the session lasts)

    const Apify = require('apify');

    Apify.main(async () => {
        const requestList = await Apify.openRequestList('my-list', ['http://www.example.com']);
        const proxyConfiguration = await Apify.createProxyConfiguration({
            groups: ['RESIDENTIAL'],
            countryCode: 'US'
        });

        const crawler = new Apify.PuppeteerCrawler({
            requestList,
            proxyConfiguration,
            useSessionPool: true,
            sessionPoolOptions: {
                sessionOptions: { maxPoolSize: 1 }, // Using single session
            },
            handlePageFunction: async ({ page, request }) => {
                return Apify.pushData({ title: await page.title() });
            },
            handleFailedRequestFunction: ({ request }) => {
                console.error('Request failed', request.url, request.errorMessages);
            },
        });

        await crawler.run();
    });

Create a new session with IP from GB for each browser launched during the Crawler run.

    const Apify = require('apify');

    Apify.main(async () => {
        const requestList = await Apify.openRequestList('my-list', ['http://www.example.com']);
        const proxyConfiguration = await Apify.createProxyConfiguration({
            groups: ['RESIDENTIAL'],
            countryCode: 'GB'
        });

        const crawler = new Apify.PuppeteerCrawler({
            requestList,
            proxyConfiguration,
            useSessionPool: true,
            handlePageFunction: async ({ page, request }) => {
                return Apify.pushData({ title: await page.title() });
            },
            handleFailedRequestFunction: ({ request }) => {
                console.error('Request failed', request.url, request.errorMessages);
            },
        });

        await crawler.run();
    });

## [](#usage-in-apify-launchPuppeteer) Usage in [Apify.launchPuppeteer()](https://sdk.apify.com/docs/api/apify#apifylaunchpuppeteeroptions)

Use a single IP from Germany for all requests done in the launched browser

    const Apify = require('apify');

    Apify.main(async () => {
        const proxyConfiguration = await Apify.createProxyConfiguration({
            groups: ['RESIDENTIAL'],
            countryCode: 'DE'
        });
        const proxyUrl = proxyConfiguration.newUrl('my_session1');

        const browser = await Apify.launchPuppeteer({
            proxyUrl
        });

        const page = await browser.newPage();

        await page.goto('http://www.example.com');

        const html = await page.content();

        console.log('HTML:');
        console.log(html);
    });

## [](#usage-with-request) Usage with our [requestAsBrowser](https://sdk.apify.com/docs/api/utils#utilsrequestasbrowseroptions)

Make a request with Residential Proxy using the [request](https://www.npmjs.com/package/request) NPM package

    const Apify = require('apify');

    Apify.main(async () => {
        const proxyConfiguration = await Apify.createProxyConfiguration({
            groups: ['RESIDENTIAL'],
        });
        try {
            const { body } = await Apify.utils.requestAsBrowser({
                url: 'https://www.example.com',
                proxyUrl: proxyConfiguration.newUrl(),
            });
            console.log(body); // returns HTML of returned page
        } catch (e) {
            console.error(e);
        }
    });


Use the same IP for two requests with proxy geolocated in France using our [requestAsBrowser](https://sdk.apify.com/docs/api/utils#utilsrequestasbrowseroptions).

    const Apify = require('apify');

    Apify.main(async () => {
        const proxyConfiguration = await Apify.createProxyConfiguration({
            groups: ['RESIDENTIAL'],
            countryCode: 'FR',
        });
        const proxyUrl = proxyConfiguration.newUrl('my_session');

        try {
            const response1 = await Apify.utils.requestAsBrowser({
                url: 'https://api.apify.com/v2/browser-info',
                proxyUrl,
                json: true
            });
            const response2 = await Apify.utils.requestAsBrowser({
                url: 'https://api.apify.com/v2/browser-info',
                proxyUrl,
                json: true
            });
            console.log(response1.body.clientIp);
            console.log('should be the same as');
            console.log(response2.body.clientIp);
        } catch (e) {
            console.error(e);
        }
    });




