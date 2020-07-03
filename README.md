# GNR Comparo

Compare your run with the Great North Run! See how far through you got. Which of your local landmarks match up to iconic things on the route? Maybe passing the corner shop is like crossing the Tyne Bridge?

This app was built to help the [Team 40x40](https://fortybyforty2019.com/) runners train and raise money for the GNR 2019.

Try it out here: [http://gnrcomparo.appspot.com](https://gnrcomparo.appspot.com)

In 2019 it was built with Laravel & Vue. I'd been using these for work and enjoying the mix, and wanted to try it out for prototyping a brand new app from scratch. In the end, it only used Laravel as a front controller to the Vue build.

In 2020, I have split the Vue code out of Laravel/laravel-mix and repackaged it as a standalone Vue app. Laravel has been replaced by Lumen. Google App Engine only allows one app entry point, so Lumen still works as a simple front controller to point to the Vue build. Lumen should still provide enough of a framework to coordinate the Strava API interaction and provide endpoints for that back to the Vue app.

I deployed it on Google App Engine (PHP7 standard environment) because:

- it has a generous free usage tier (in the standard environment)
- it provides a handy domain name
- I use it already for [Simon Likes Maps](https://www.simonlikesmaps.com) with the Python2.7 runtime and I wanted to try out the PHP runtime

And I've made it open source because it's a good example of my work with PHP & Vue and maps. These are technologies I really enjooy working with, so it makes a good portfolio project for people to have a look at the code I can produce.
