# middleware axios AbortSignal memory leak

## Context

A memory leak has been identified when `AbortSignal` is used with `axios` in `middlware.ts`. This issue is not present when using `fetch` in `middleware.ts` or when using `axios` with `AbortSignal` in pages.  
It seems that this was introduced in Next 15.4. This repo uses 15.5.5, and the issue is also present in canary 16.

## Repo overview

This repo was created using `npx create-next-app@latest` with defaults selected.
The repo also contains 4 pages that fetch data from a mock server and display some placeholder text on the screen.
The mock server is a basic hello world `express` app. I chose to use an external app to avoid the next app calling itself.

Two of the pages - `/axios` and `/fetch` are server components which fetch data within the component, both using an `AbortSignal`
The other two pages - `/middleware/axios` and `/middleware/fetch` trigger some fetching to happen in `middleware.ts`.

All 4 places that data fetching occurs uses either `axios` or `fetch` with an `AbortSignal` provided, and the make a call to the mock server.

These 3 pages do not increase memory: `/axios`, `/fetch`, `/middleware/fetch`.
The 4th page, `/middleware/axios` does cause a memory leak.

The start script has heapsnapshot dumping enabled via `NODE_OPTIONS="--heapsnapshot-signal=SIGUSR2"`

## Reproduction steps

1. Run `npm install`
2. Build the next app: `npm run build`
3. Run the mock server in a separate terminal window: `node mockServer.js`
4. Start the next app: `npm run start`
5. Make many requests (I do 1,000) to a single page
6. Take a heap snapshot using `kill -USR2 <next-server PID>`. You can find the `PID` by running `ps aux | grep "next-server (v15.5.5)"`
7. Open the heap snapshot in chrome dev tools and look for increasing memory
8. Repeat steps 5-7 with all other pages.

You should notice that `/middleware/axios` is the only one that increases memory. You can also verify this by leaving the server running for a little bit and noticing that the memory is never returned.
