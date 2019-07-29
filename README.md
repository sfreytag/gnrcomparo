# GNR Comparo

Compare your run with the Great North Run! See how far through you got. Which of your local landmarks match up to iconic things on the route? Maybe passing the corner shop is like crossing the Tyne Bridge?

This app was built to help the [Team 40x40](https://fortybyforty2019.com/) runners train and raise money for the GNR 2019.

Try it out here: [http://gnrcomparo.appspot.com](https://gnrcomparo.appspot.com)

It's built with Laravel & Vue. I'd been using these for work and enjoying the mix, and wanted to try it out for prototyping a brand new app from scratch. It has loads of project generation scaffolding to get something up and running very quickly. This current version does not make that much use of Laravel itself, but nonetheless the process seemed worthwhile: I got server-side routing and webpack for free and it's nice knowing there's flexibility to do other stuff in future. It can always be stripped back in future when I know what if anything will be added to the app.

The one thing I really do want to add is Strava integration to help compare your Strava route to the GNR without downloading the GPX. It'll be interesting to learn the Strava APIs.

I deployed it on Google App Engine (php7 standard environment) because:

- it has a generous free usage tier (in the standard environment)
- it provides a handy domain name
- I use it already for [Simon Likes Maps](https://www.simonlikesmaps.com) with the Python2.7 runtime and I wanted to try out the PHP runtime

And I've made it open source because it's a good example of my work with Laravel & Vue, and maps. These are technologies I really enjooy working with, so it makes a good portfolio project for people to have a look at the code I can produce.
