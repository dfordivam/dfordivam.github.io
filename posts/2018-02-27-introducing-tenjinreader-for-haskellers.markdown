---
title: Introducing tenjinreader.com for Haskellers
---

I created this application based on my own experiences and requirements for Japanese language learning.

This post is mainly for Haskell developers, so I will not discuss anything about the Japanese language learning part of this application here. Checkout my other post for details on that.

***

## The Big Picture

The app is written end-to-end in Haskell. 

Moreover, I tried to use a bunch of new (experimental) stuff, so this blog post is about my experience 

I started web programming just one year back. I have used Haskell for more than 4 years, and it is easier for me to use complex Haskell stuff than learn javascript.

In a period of almost 5 months I was able to make this app from an idea to a beta release.

The total code base is approx (lines of Haskell code).
[Source code on Github](https://github.com/blueimpact/tenjinreader)

- Frontend - 3k
- Backend - 3k
- Common - 1k

and about 1k more for some extra libraries I created for this project, but pulled in a separate project.

## Reflex FRP

Frontend is written entirely using Reflex-DOM FRP library.

#### The good

* Its Haskell

  Having both FE and BE in Haskell is awesome. There have been many occasions where I have moved the code (by cut-paste) between the three projects.

* With Reflex code refactoring and changes in UI are very easy. Moving around widgets, re-using widgets from multiple places, its all very easy.

* [`reflex-project-skeleton`](https://github.com/ElvishJerricco/reflex-project-skeleton), `nix`, and `jsaddle-warp` are amazing for development workflow. I even do the deployment via `nix-copy-closure` which is awesome.

  There was an initial struggle with nix, but there are now a number of good resources, and #reflex-frp is an awesome support channel.

* For websocket I created a [small library](https://github.com/dfordivam/reflex-websocket-interface) specially for reflex. Behind the scenes this library does a lot of plumbing of Events, encoding / decoding of messages.

  The experience has been great, I dont have to think a second time before changing some message type or adding new requests.
  
  Moreover the code to communicate with server can now be part of the widget itself. Things works so seamlessly.
  
  In the start of a new feature I dont really know what all data will be required in frontend, so I pull-in the complete entries from DB (thats why a lot of DB stuff is in `common` package with ToJSON and FromJSON instances).

  Then I write code in frontend to manipulate the data and get it to what I need. Later I have the choice to move this code to backend, and only send the manipulated data to frontend. (But many times I let things as it is, the client should also do some work!)

I am sure there is a lot more good, but I think it would be obvious in a comparison to other technologies. I hardly have any experience of frontend development, so I will now start with the pain points...

#### Difficulties

* Large `rec` blocks are hard to compile as the error messages can be misleading, especially if you make mistakes in using things which are to be used monadically `MonadWidget` with `Event`/ `Dynamic`.

  The `rec` block can also introduce strange loops or hang the app. Although they are not that hard to debug (as you can fairly easily pin point the code causing it). But the solution often involves heuristically adding `delay` to make it work.

  Tip 1 - Dont re-use names like this, its very easy to mess things up and create loops.
  ```
  rec
    retVal <- do
      retVal <- someStuff
      ...
      return retVal
  ```
  
  Tip 2 -  When the code starts to get complicated with nested `rec` blocks, create a separate function.
  
  There is also a weird problem that a polymorphic function does not type-check if defined inside a `let` block inside `rec` (without an explicit type signature).
  This can be annoying for a new comer, as the code is fine but still it does not compile.

* The `Reflex.Collection` consist of a bunch of useful widgets, but their behaviour and use-case can be dificult to comprehend.

* It can be tricky to handle big `Dynamic` values.

  For example I had to create a `Event t (Set Int)` from `[Dynamic t (Bool, Int)]`. The `[]` -> `Set` -> `tagDyn` approach was not good, instead `[]` -> `tagDyn` -> `Set` is better.

  `dyn` is tricky, avoid it. Use `widgetHold` instead.
  
* Reflex eco-system still need a consolidated resource of information. I have done some contributions last year to the docs, and have a bunch of more stuff lying around in my local repo to update the docs.

* When opening the app, about 10% of the time there are some strange errors / assertion in reflex runtime like "Causality loop found". On doing a refresh it mostly works fine.

* The performance of certain parts of app is terrible, I attribute it to `ghcjs` mostly (as the performance was better with `jsaddle-warp`), and hope with `webghc` things will be much better.

* When the structure of DOM is not according to what you desire, you have the choice to refactor code and pass `Event` and `Dynamic` all around the app, or to use some CSS magic to fix stuff (like `z-index`, `position: absolute`).

* When using external JS libraries with reflex-dom

  * Delay in events are required to get things like DOM Apis / FFI to work properly.

  * If the external JS throws an exception then the app is dead. Perhaps there is a way to catch and continue?

  * Integration with UI libraries like Bootstrap, Semantic UI, etc is still in development, more effort is required to make it work.
  
* The use of a monadic style for DOM creation can be difficult at times, but I feel its a minor thing and the pros outweigh the cons.

## Haskey for persistent DB

For the DB I have taken an even more experimental approach by not using any conventional DB or even acid libraries.
  The reason for this was I wanted simplicity in usage, and the data should not be completely in memory.
  
The [`haskey`](https://github.com/haskell-haskey/haskey) library released in late last year fortunately had both these features.
  
This was a very important for me to do fast development, as I re-modelled the schema dozens of times based on the requirements in Frontend, and incremental addition of small features here and there.

So `haskey` is a new library and **definitely not yet ready for production.**

While still developing the application I found a [major bug](https://github.com/haskell-haskey/haskey/issues/70) in its code, it was fixed fast thanks to the author!

But the real scary stuff was; after putting the site on production my own data got corrupted after a few days. I dont know yet what happened; fortunately I had backups to recover it from the previous day.

So there are definitely more bugs lurking around in the code, and the only safe way to use this library is to always take backups / snapshots (like hourly)

One good thing I did with `haskey` is to keep the data of every user in a separate DB (filesystem folder). This greatly simplified my code base and doing recovery from backups.

#### Migrations in Haskey

For migrations I used a method inspired by *Trees that Grow* paper. The idea is to use `type family` to have different variations of a tree data structure. In my case I have just two variations of the main schema: a `CurrentDB` and an `OldDB`. The application code always works on the `CurrentDB` and the `OldDB` is used to for migrating old schema to the current one.

Maintaining many versions of schema will be a much more complicated task, perhaps it can be done using this approach but I am not too sure.

There were a few issues in getting the code to compile with the use of `type family`, specifically in the `instance` / `deriving instance` declarations. I have used some hit and trial to get it working. But it would be good to have a better understanding / proper way of doing it. (ie dont refer to my code as a correct way to do it, there might be bugs)

### Yesod

The backend is using `yesod`, it is very simple to set up a web site with authentication using the existing libraries. The major work is done via websocket, so there was no need for a library like `servant`.


## Dev workflow
  - Spacemacs with simple support of haskell like syntax highlighting, and hindent
  - Local hoogle server running from nix-shell
  - `./cabal new-repl` from the project skeleton, and sometimes ghcid.
  - Frontend running on Chrome via `jsaddle-warp`.

## Final thoughts

  - Your application need to have a certain level of complexity to demand the use of Reflex. I think this app had a good enough complexity to deserve this.

    There is a learning curve with reflex. It will only pay for itself when the application is complex.

  - `Haskey` is a good library for fast prototyping, more robust DB can be put in place later.

  - I embraced `lens` (and some of its operators) while working on this, I would say that the effort pays back.

  - I dont have any testing stuff right now for this app. There were a few functions which were algorithmically complex, and I just created a small test function next to it, for manual testing.

    Things have been ok as I am the only dev working on this project. With multiple people there is definitely a need for a test-suite.


