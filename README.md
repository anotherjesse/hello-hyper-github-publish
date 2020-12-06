# Svelte: Github commit -> published to hyperdrive

This is a simple proof of concept of publishing a compiled svelte app to a hyperdrive upon commit.

The contained github action will build and then publish to hyper://e89224ea508fedaa196396a4b8f326da5e0e9e4cc34ddb5e382872fbb020aaee

We will use:

 - svelte repl / https://svelte.dev/repl - to bootstrap our app
 - hyperdrive-publisher - https://github.com/RangerMauve/hyperdrive-publisher - to create / update our hyperdrive
 - dat-store - https://github.com/datproject/dat-store - to peer/pin our hyperdrive
 - beakerbrowser - https://beakerbrowser.com - to view our hyperdrive & app

## Step 0: Setting up dat-store

`hyperdrive-publisher` creates and updates hyperdrives, but you will need to persist the drive in at least one peer.

I recommend using [dat-store](https://github.com/datproject/dat-store) on a server (raspberry pi or tiny cloud vm).

I'm running my instance of `dat-store` on a google cloud vm with the neccessary udp ports exposed

## Step 1: Create a new hyperdrive

To create a hyperdrive that you will publish to using github actions, you will run the command `hyperdrive-publisher create` (either on your local system or a remote server).

    $ hyperdrive-publisher create
    Seed:
    c343220bd25b2c88cf313f349761b8347bd31c3002ba123d29b88d4ea755eff5
    URL:
    hyper://792f244134c0956c6ff26d1de88cff0a793cf52561ec0e2fd526e1e48d23bc76
    Initializing Hyperdrive
    Please add this URL to a pinning service like dat-store to continue 
   
I then go to my cloud instance and add that hyper url to `dat-store` peering list:

    node bin.js add hyper://792f244134c0956c6ff26d1de88cff0a793cf52561ec0e2fd526e1e48d23bc76

Back on my local system, I see that my new hyperdrive has been peered!

    Connected {
      remoteAddress: '::ffff:10.128.0.3',
      remoteType: 'tcp',
      remotePublicKey: <Buffer 9b f2 8b 5e f2 2d e9 7b 62 92 e3 68 0d 1e 98 16 75 55 c1 1c f2 ba 36 15 27 ee f4 a5 e4 8a 3c 5a>
    }
    Waiting to sync metadata
    Synced
    You can sync a folder with:
    hyperdrive-publisher sync c343220bd25b2c88cf313f349761b8347bd31c3002ba123d29b88d4ea755eff5

At this point the hyperdrive should be accessible (from Beaker or other hyper clients) but will be empty except for an `index.json` file.

NOTE: you need to save the `seed`!

## Step 2: publishing to hyperdrive via github actions

We can create a new github repository and add an action by copying [publish.yml from RangerMauve's example](https://github.com/RangerMauve/hyperdrive-publisher-example/blob/default/.github/workflows/publish.yml)

By default the sample publishes all the files from a checkout of the git repository.  To publish only the generated archive, make a small change to the action by adding `public/`" to end of the line:

    - name: Run Publisher
      run: npx hyperdrive-publisher sync ${{ secrets.PUBLISHER_KEY }} public/

Additionally on github you will need to go to the repository settings and create a secret called `PUBLISHER_KEY` and paste in the `seed` that was provided in Step 1.

## Step 3: Svelte App

This is the simplest svelte app, which was created by going to https://svelte.dev/repl and changing name to `svelte` then clicking download.

Place the downloaded files in the root of the git repository, then we must add a step to install the dependencies and compile the svelte app.

Adding the following steps to the action before the hyperdrive-publisher step:

    - name: install app deps
      run: npm i
    - name: compile app
      run: npm run build

At this point pushing to github should trigger the action.  Seconds later you should see the completed application in your hyper url in Beaker!
