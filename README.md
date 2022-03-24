This is a Next.js project bootstrapped with create-next-app.

Getting Started
First, run the development server:

npm run dev
# or
yarn dev
Open http://localhost:3000 with your browser to see the result.

You can start editing the page by modifying pages/index.js. The page auto-updates as you edit the file.

API routes can be accessed on http://localhost:3000/api/hello. This endpoint can be edited in pages/api/hello.js.

The pages/api directory is mapped to /api/*. Files in this directory are treated as API routes instead of React pages.

Learn More
To learn more about Next.js, take a look at the following resources:

Next.js Documentation - learn about Next.js features and API.
Learn Next.js - an interactive Next.js tutorial.
You can check out the Next.js GitHub repository - your feedback and contributions are welcome!

Deploy on Vercel
The easiest way to deploy your Next.js app is to use the Vercel Platform from the creators of Next.js.

Check out our Next.js deployment documentation for more details.

The Chainlist
thechainlist.io

  9  components/chain/chain.js 
@@ -1,4 +1,4 @@
import React, { useState, useEffect } from 'react';
import React, { useState, useEffect,useMemo } from 'react';
import { Typography, Paper, Grid, Button, Tooltip } from '@material-ui/core'
import Skeleton from '@material-ui/lab/Skeleton';
import { useRouter } from 'next/router'
@@ -8,6 +8,7 @@ import classes from './chain.module.css'

import stores from '../../stores/index.js'
import { getProvider } from '../../utils'
import { icons } from "../../utils/icons";

import {
  ERROR,
@@ -88,6 +89,10 @@ export default function Chain({ chain }) {
    }

  }
  const icon = useMemo(() => {
    const chainName = chain.name.toLowerCase().split(" ")[0];
    return (chain.icon && icons[chain.icon]) || (chainName && icons[chainName]) || "/chains/unknown-logo.png";
  }, [chain]);

  if(!chain) {
    return <div></div>
@@ -97,7 +102,7 @@ export default function Chain({ chain }) {
    <Paper elevation={ 1 } className={ classes.chainContainer } key={ chain.chainId }>
      <div className={ classes.chainNameContainer }>
        <img
          src='/connectors/icn-asd.svg'
          src={icon}
          onError={e => {
            e.target.onerror = null;
            e.target.src = "/chains/unknown-logo.png";
  2  components/chain/chain.module.css 
@@ -30,6 +30,8 @@

.avatar {
  margin-right: 24px;
  border: 0px solid;
  border-radius: 50%;
}

.chainNameContainer {
 3  components/shutdownNotice/package.json 
This file was deleted.

 32  components/shutdownNotice/shutdownNotice.js 
This file was deleted.

 130  components/shutdownNotice/shutdownNotice.module.css 
This file was deleted.

  4  components/snackbar/snackbarController.jsx 
@@ -29,12 +29,12 @@ class SnackbarController extends Component {
    }
  }

  componentWillMount() {
  componentDidMount() {
    emitter.on(ERROR, this.showError);
    emitter.on(TX_SUBMITTED, this.showHash);
  }

  componentWillUnmount() {
  componentDidUnmount() {
    emitter.removeListener(ERROR, this.showError);
    emitter.removeListener(TX_SUBMITTED, this.showHash);
  };
 34  oldREADME.md 
@@ -0,0 +1,34 @@
This is a [Next.js](https://nextjs.org/) project bootstrapped with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app).

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `pages/index.js`. The page auto-updates as you edit the file.

[API routes](https://nextjs.org/docs/api-routes/introduction) can be accessed on [http://localhost:3000/api/hello](http://localhost:3000/api/hello). This endpoint can be edited in `pages/api/hello.js`.

The `pages/api` directory is mapped to `/api/*`. Files in this directory are treated as [API routes](https://nextjs.org/docs/api-routes/introduction) instead of React pages.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js/) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/deployment) for more details.
 6,630  package-lock.json 
Large diffs are not rendered by default.

  8  package.json 
@@ -7,16 +7,16 @@
    "build": "next build",
    "export": "next export",
    "build2": "next build && next export",
    "start": "next start"
    "start": "next start -p $PORT"
  },
  "dependencies": {
    "@material-ui/core": "^4.11.3",
    "@material-ui/icons": "^4.11.2",
    "@material-ui/lab": "^4.0.0-alpha.57",
    "flux": "^4.0.1",
    "next": "10.0.7",
    "react": "17.0.1",
    "react-dom": "17.0.1",
    "next": "^12.1.0",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "swr": "^0.5.6",
    "web3": "^1.3.4"
  }
  30  pages/_app.js 
@@ -1,9 +1,9 @@
import React, { useState, useEffect } from 'react';
import Script from 'next/script';
import { ThemeProvider } from '@material-ui/core/styles';
import CssBaseline from '@material-ui/core/CssBaseline';

import SnackbarController from '../components/snackbar'
import ShutdownNotice from '../components/shutdownNotice'

import stores from '../stores/index.js'

@@ -35,20 +35,32 @@ function MyApp({ Component, pageProps }) {
    stores.dispatcher.dispatch({ type: CONFIGURE })
  },[]);

  const [shutdownNoticeOpen, setShutdownNoticeOpen] = useState(true);
  const closeShutdown = () => {
    setShutdownNoticeOpen(false)
  }

  return (
  <>
    <Script
      strategy="afterInteractive"
      src={`https://www.googletagmanager.com/gtag/js?id=G-G9KYFJHW21`}
      />
    <Script
      id="gtag-init"
      strategy="afterInteractive"
      dangerouslySetInnerHTML={{
        __html: `
          window.dataLayer = window.dataLayer || [];
          function gtag(){dataLayer.push(arguments);}
          gtag('js', new Date());
          gtag('config', 'G-G9KYFJHW21', {
            page_path: window.location.pathname,
          });
        `,
      }}
    />
    <ThemeProvider theme={ themeConfig }>
      <CssBaseline />
      <Component {...pageProps} changeTheme={ changeTheme } />
      <SnackbarController />
      { shutdownNoticeOpen &&
        <ShutdownNotice close={ closeShutdown } />
      }
    </ThemeProvider>
  </>
  )
}

  36  pages/index.js 
@@ -1,4 +1,5 @@
import React, { useState, useEffect } from 'react';
import Script from 'next/script'
import Head from 'next/head'
import { useRouter } from 'next/router'
import styles from '../styles/Home.module.css'
@@ -126,6 +127,25 @@ function Home({ changeTheme, theme }) {
      <Head>
        <title>Chainlist</title>
        <link rel="icon" href="/favicon.png" />
        {/* <Script
        strategy="afterInteractive"
        src={`https://www.googletagmanager.com/gtag/js?id=G-G9KYFJHW21`}
        />
        <Script
          id="gtag-init"
          strategy="afterInteractive"
          dangerouslySetInnerHTML={{
            __html: `
              window.dataLayer = window.dataLayer || [];
              function gtag(){dataLayer.push(arguments);}
              gtag('js', new Date());
              gtag('config', 'G-G9KYFJHW21', {
                page_path: window.location.pathname,
              });
            `,
          }}
        /> */}

      </Head>

      <main className={styles.main}>
@@ -146,13 +166,22 @@ function Home({ changeTheme, theme }) {
                <Typography className={ classes.buttonLabel }>Add Your Network</Typography>
              </Button>
              <div className={ classes.socials }>
                <a className={ `${classes.socialButton}` } href='https://github.com/antonnell/networklist-org.git' target='_blank' rel="noopener noreferrer" >
                <a className={ `${classes.socialButton}` } href='https://github.com/izzetemredemir/networklist-org' target='_blank' rel="noopener noreferrer" >
                  <svg version="1.1" width="24" height="24" viewBox="0 0 24 24">
                    <path fill={ '#2F80ED' } d="M12,2A10,10 0 0,0 2,12C2,16.42 4.87,20.17 8.84,21.5C9.34,21.58 9.5,21.27 9.5,21C9.5,20.77 9.5,20.14 9.5,19.31C6.73,19.91 6.14,17.97 6.14,17.97C5.68,16.81 5.03,16.5 5.03,16.5C4.12,15.88 5.1,15.9 5.1,15.9C6.1,15.97 6.63,16.93 6.63,16.93C7.5,18.45 8.97,18 9.54,17.76C9.63,17.11 9.89,16.67 10.17,16.42C7.95,16.17 5.62,15.31 5.62,11.5C5.62,10.39 6,9.5 6.65,8.79C6.55,8.54 6.2,7.5 6.75,6.15C6.75,6.15 7.59,5.88 9.5,7.17C10.29,6.95 11.15,6.84 12,6.84C12.85,6.84 13.71,6.95 14.5,7.17C16.41,5.88 17.25,6.15 17.25,6.15C17.8,7.5 17.45,8.54 17.35,8.79C18,9.5 18.38,10.39 18.38,11.5C18.38,15.32 16.04,16.16 13.81,16.41C14.17,16.72 14.5,17.33 14.5,18.26C14.5,19.6 14.5,20.68 14.5,21C14.5,21.27 14.66,21.59 15.17,21.5C19.14,20.16 22,16.42 22,12A10,10 0 0,0 12,2Z" />
                  </svg>
                  <Typography variant='body1' className={ classes.sourceCode }>View Source Code</Typography>
                </a>
                <Typography variant='subtitle1' className={ classes.version }>Version 1.0.7</Typography>
                <Typography variant='subtitle1' className={ classes.version }>Original project by @antonnell</Typography>
              </div>
              <div className={ classes.socials }>
                <a className={ `${classes.socialButton}` } href='https://twitter.com/thechainlist' target='_blank' rel="noopener noreferrer" >
                  <svg version="1.1" width="24" height="24" viewBox="0 0 410.155 410.155">
                    <path xmlns="http://www.w3.org/2000/svg" fill={ '#2F80ED' } d="M403.632,74.18c-9.113,4.041-18.573,7.229-28.28,9.537c10.696-10.164,18.738-22.877,23.275-37.067  l0,0c1.295-4.051-3.105-7.554-6.763-5.385l0,0c-13.504,8.01-28.05,14.019-43.235,17.862c-0.881,0.223-1.79,0.336-2.702,0.336  c-2.766,0-5.455-1.027-7.57-2.891c-16.156-14.239-36.935-22.081-58.508-22.081c-9.335,0-18.76,1.455-28.014,4.325  c-28.672,8.893-50.795,32.544-57.736,61.724c-2.604,10.945-3.309,21.9-2.097,32.56c0.139,1.225-0.44,2.08-0.797,2.481  c-0.627,0.703-1.516,1.106-2.439,1.106c-0.103,0-0.209-0.005-0.314-0.015c-62.762-5.831-119.358-36.068-159.363-85.14l0,0  c-2.04-2.503-5.952-2.196-7.578,0.593l0,0C13.677,65.565,9.537,80.937,9.537,96.579c0,23.972,9.631,46.563,26.36,63.032  c-7.035-1.668-13.844-4.295-20.169-7.808l0,0c-3.06-1.7-6.825,0.485-6.868,3.985l0,0c-0.438,35.612,20.412,67.3,51.646,81.569  c-0.629,0.015-1.258,0.022-1.888,0.022c-4.951,0-9.964-0.478-14.898-1.421l0,0c-3.446-0.658-6.341,2.611-5.271,5.952l0,0  c10.138,31.651,37.39,54.981,70.002,60.278c-27.066,18.169-58.585,27.753-91.39,27.753l-10.227-0.006  c-3.151,0-5.816,2.054-6.619,5.106c-0.791,3.006,0.666,6.177,3.353,7.74c36.966,21.513,79.131,32.883,121.955,32.883  c37.485,0,72.549-7.439,104.219-22.109c29.033-13.449,54.689-32.674,76.255-57.141c20.09-22.792,35.8-49.103,46.692-78.201  c10.383-27.737,15.871-57.333,15.871-85.589v-1.346c-0.001-4.537,2.051-8.806,5.631-11.712c13.585-11.03,25.415-24.014,35.16-38.591  l0,0C411.924,77.126,407.866,72.302,403.632,74.18L403.632,74.18z"/>
                  </svg>
                  <Typography variant='body1' className={ classes.sourceCode }>Follow us on Twitter</Typography>
                </a>
                <Typography variant='subtitle1' className={ classes.version }>@thechainlist</Typography>
              </div>
            </div>
          </div>
@@ -185,7 +214,8 @@ function Home({ changeTheme, theme }) {
              <Header changeTheme={ changeTheme } />
            </div>
            <div className={ classes.cardsContainer }>
              { hideMultichain === '0' && <MultiChain closeMultichain={ closeMultichain } /> }
              {/* { hideMultichain === '0' && <MultiChain closeMultichain={ closeMultichain } /> } 
                to use the ad uncomment this expression */}
              {
                data && data.filter((chain) => {
                  if(search === '') {
