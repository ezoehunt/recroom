# Rec Room Documentation

![Rec Room logo](images/recroom-logo.jpg?raw=true)


# Build Your First App

We’ll create a simple clock app with (minimal) time zone support for our example. The app will let you have a clock and compare it with a few time zones.

## Create the App
1. Change to the directory where you want to create your project.
2. Create a new app called "world-clock."
   
   ```
   $ recroom new world-clock
   ```
3. Change directory to the new project folder.
   
   ```
   $ cd world-clock
   ```
4. Run the new app in your default browser.
   
   ```  
   $ recroom run
   ```

### Add the Current Time

First, we’ll add the current time to the main tab. Rec Room supports Ember’s MVC app structure, but also offers simple "pages" for a controller without a 1:1 relationship to a model. 

1. Generate a new page that will show our clock.

   ```
   $ recroom generate page Clock
   ```
2. This will create a new template at app/templates/clock.hbs. Now let’s change clock.hbs to include the variable that will output our local time. Open app/templates/clock.hbs and add the following:

   ```
   <h2>Local Time: {{localTime}}</h2>
   ```
3. Now let's add that variable to our ClockController. Open app/scripts/controllers/clock_controller.js and add the following:

   ```
   WorldClock.ClockController = Ember.ObjectController.extend({
      localTime: new Date().toLocaleTimeString()
   });
   ```
You can see that any property inside the controller is accessible inside that controller’s template. We've define the 1ocalTime property, and it's carried into our template context.

4. Open http://localhost:9000/#clock in your browser to see the clock. Of course, it just shows the time it was when the controller was initialized; right now, there is no live updating of the time. 
5. Let's update the time every second inside the controller. Open app/scripts/controllers/clock_controller.js and add the following:
   ```
   WorldClock.ClockController = Ember.ObjectController.extend({
      init: function() {
         // Update the time.
         this.updateTime();
 
      // Run other controller setup.
         this._super();
      },
 
      updateTime: function() {
         var _this = this;
 
         // Update the time every second.
         Ember.run.later(function() {
               _this.set('localTime', new Date().toLocaleTimeString());
               _this.updateTime();
         }, 1000);
      },
 
      localTime: new Date().toLocaleTimeString()
   });
   ```
6. Open http://localhost:9000/#clock in your browser to see that our clock automatically updates every second. 

With this simple clock app, we've demonstrated how to work with Ember’s data-binding between controllers and templates. If you change a value in a controller, model, or view that’s wired up to a template, the template will automatically change that data for you.

### Adding Timezones

Next, let's add a few timezones that a person can add to their own collection of timezones to compare against local time. This will help a person schedule their meetings with friends in San Francisco, Buenos Aires, and London. 

1. Rec Room let's create a timezone model (and accompanying controllers/routes/templates) with the same generate command, but this time we’ll just generate the model:.

   ```
   $ recroom generate model Timezone
   ```
2. We want each timezone included in our app to have a name and an offset value. So we'll add them as model attributes, and we'll use Ember Data for this. Open app/scripts/models/timezone_model.js and add the following:

   ```
   WorldClock.Timezone = DS.Model.extend({
      name: DS.attr('string'),
      offset: DS.attr('number')
   });
  ```
3. Next we want to let a person choose from a list of all timezones. For this we’ll use [Moment Timezone](), an awesome JavaScript library for dealing with dates and times in JavaScript.  Install Moment Timezone using [bower]().

   ```
   $ bower install moment-timezone --save
   ```
4. Open app/index.html and add the following.

   ```
<!-- build:js(app) scripts/components.js -->
<!-- [Other script tags] -->
<script src="bower_components/moment/moment.js"></script>
<script src="bower_components/moment-timezone/builds/moment-timezone-with-data-2010-2020.js"></script>
<!-- endbuild -->
   ```
Adding that tag automatically adds moment-timezone-with-data-2010-2020.js to our built app. 

5. Next we'll add a tab to the page that lets people edit their timezones on a different screen than their clocks. To add a tab, we just need to open app/templates/application.hbs and add a tab. While we’re there, let's change the main tab from the useless {{#linkTo 'index'}} and point it to {{#linkTo 'clock'}}. Your updated application.hbs should look like this:

   ```
<x-layout>
  <header>
    <x-appbar>
      <h1>{{t app.title}}</h1>
    </x-appbar>
  </header>
  <section>
    {{outlet}}
  </section>
  <footer>
    <x-tabbar>
      <x-tabbar-tab>
        {{#link-to 'clock'}}Clock{{/link-to}}
      </x-tabbar-tab>
      <x-tabbar-tab>
        {{#link-to 'timezones'}}Timezones{{/link-to}}
      </x-tabbar-tab>
    </x-tabbar>
  </footer>
</x-layout>
   ```

6. Now open http://localhost:9000. in your browser. Notice how the root URL points to a useless welcome page? We probably want the default route to be our ClockController. So let's set the index route to redirect to it. Open app/scripts/routes/application_route.js and add the following:
   ```
WorldClock.ApplicationRoute = Ember.Route.extend({
   redirect: function() {
      this.transitionTo('clock');
   }
});
   ```

### Interacting with Timezone Models

We’ll keep things simple for our example and allow users to select a timezone from a <select> tag and add it with a button. It will show up in their list of timezones, and they can delete it if they want from there. The clock tab will show all times. 

1. Open app/scripts/controllers/timezones_controller.js. Frst, we’ll add our timezone data from Moment.js into our TimezonesController.  Then we'll implement two actions: “add” and “remove”. These will be used in our template, which should look like this:

   ```
WorldClock.TimezonesController = Ember.ObjectController.extend({
    init: function() {
        var timezones = [];
 
        for (var i in moment.tz._zones) {
          timezones.push({
              name: moment.tz._zones[i].name,
              offset: moment.tz._zones[i].offset[0]
          });
      }
 
      this.set('timezones', timezones);
 
      this._super();
  },
 
  selectedTimezone: null,
 
  actions: {
      add: function() {
          var timezone = this.store.createRecord('timezone', {
              name: this.get('selectedTimezone').name,
              offset: this.get('selectedTimezone').offset
          });
 
          timezone.save();
      },
 
      remove: function(timezone) {
          timezone.destroyRecord();
      }
  }
});
   ```
So we created a list of all available timezones with offsets. Then we added methods that allow us to add or remove timezones from our offline data store. 

2. Next we'll modify the timezones template to use the actions and variables we just created. Open app/templates/timezones.hbs. Use the Ember SelectView and the {{action}} helper to call our add and remove methods. Your file should look like this:

   ```
<h2>Add Timezone</h2>
 
<p>{{view Ember.Select content=timezones selection=selectedTimezone
       optionValuePath='content.offset' optionLabelPath='content.name'}}</p>
 
<p><button {{action add}}>Add Timezone</button></p>
 
<h2>My Timezones</h2>
 
<ul>
  {{#each model}}
    <li>{{name}} <button {{action remove this}}>Delete</button></li>
  {{/each}}
</ul>
   ```

Now we have a Timezones tab that allows us to add and remove Timezones we want to track. This data persists between app refreshes. 

The last thing we need to do is show these times relative to our local time in our clock tab. To do this we need to load all the Timezone models in the ClockRoute. They’re automatically loaded in the TimezonesRoute, but it’s easy to add them in the ClockRoute.

3. Open app/scripts/routes/clock_route.js and add the following:

   ```
WorldClock.ClockRoute = Ember.Route.extend({
    model: function() {
        return this.get('store').find('timezone');
    }
});
   ```
Because of the way our Ember app is wired up, we load all our models in the route and they are sent to the controller once the data store has asynchonously loaded all of the models. The request to find('timezone') actually returns a Promise object, but Ember’s router handles the Promise resolving for us automatically so we don’t have to manage callbacks or Promises ourselves.

Now we have access to all the user’s Timezones in the ClockController, so we can make times in each timezone the user has requested and show them in a list. 

4. Let's add each Timezone’s current time to our ClockController using Moment.js. Open app/scripts/controllers/clock_controller.js and add the following:

   ```
WorldClock.ClockController = Ember.ObjectController.extend({
    updateTime: function() {
        var _this = this;
 
        // Update the time every second.
        Ember.run.later(function() {
            _this.set('localTime', moment().format('h:mm:ss a'));
 
            _this.get('model').forEach(function(model) {
                model.set('time',
                          moment().tz(model.get('name')).format('h:mm:ss a'));
            });
 
            _this.updateTime();
        }, 1000);
    }.on('init'),
 
    localTime: moment().format('h:mm:ss a')
});
   ```

5. Your final app/templates/clock.hbs should look like this:

   ```
<h2>Local Time: {{localTime}}</h2>
 
<p>
  {{#each model}}
    <h3>{{name}}: {{time}}</h3>
  {{/each}}
</p>
   ```

6. Open http://localhost:9000/ in your browser to see the your final clock app. You've got an offline app that shows time zones in various places, saves the data offline, and updates every second without you having to do much work!


