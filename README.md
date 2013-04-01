Flask FeatureFlags
===================

This lets you enable or disable features based on configuration. Very useful when you're deploying from trunk.

You can also extend this to add your own functionality, for simple a/b testing or whitelisting.


Installation
============

Installing from source is easy. Download the source code, then run this:

    $ cd Flask-FeatureFlags
    $ python setup.py install


Setup
=====

Setup is also simple:

    from flask import Flask
    from flask_featureflags import FeatureFlagExtension

    app = Flask(__name__)

    feature_flags = FeatureFlagExtension(app)

In your Flask app.config, create a ``FEATURE_FLAGS`` dictionary, and add any features you want as keys.

For example, to have 'unfinished_feature' hidden in production but active in development:

    class ProductionConfig(Config):

        FEATURE_FLAGS = {
            'unfinished_feature' : False,
        }


    class DevelopmentConfig(Config):

        FEATURE_FLAGS = {
          'unfinished_feature' : True,
        }

If a feature doesn't exist, it is assumed to be inactive.


Usage
=====

Controllers/Views
-----------------

If you want to protect an entire view:

    from flask import Flask
    import flask_featureflags as feature

    @feature.is_active_feature('unfinished_feature', redirect_to='/old/url')
    def index():
        # unfinished view code here

The redirect_to parameter is optional. If you don't specify, the url will return a 404.

If your needs are more complicated, you can check inside the view:

    from flask import Flask
    import flask_featureflags as feature

    def index():
        if feature.is_active('unfinished_feature') and some_other_condition():
            # do new stuff
        else:
            # do old stuff

Templates
---------

You can also check for features in template code:

    {% if 'unfinished_feature' is active_feature %}
        new behavior here!
    {% else %}
        old behavior...
    {% endif %}


Customization
=============

If you need custom behavior, you can write your own feature flag handler.

A feature flag handler is simply a function that takes the feature name as input, and returns True (the feature is on) or False (the feature is off).

For example, if you want to enable features only on Tuesdays:

    from datetime import date

    def is_it_tuesday(feature):
      return date.today().weekday() == 2:

You can register the handler like so:

    from flask import Flask
    from flask_featureflags import FeatureFlagExtension

    app = Flask(__name__)

    feature_flags = FeatureFlagExtension(app)
    feature_flags.add_handler(is_it_tuesday)


If you want to remove a handler for any reason, just do:

    feature_flags.remove_handler(is_it_tuesday)

If you try to remove a handler that was never added, the code will silently ignore you.

To clear all handlers (thus effectively turning all features off):

    feature_flags.clear_handlers()


Chaining multiple handlers
--------------------------

You can define multiple handlers. If any of them return true, the feature is considered on.

For example, say you want features to be enabled on Tuesdays *or* Fridays:

    feature_setup.add_handler(is_it_tuesday)
    feature_setup.add_handler(is_it_friday)

**Important:** the order of handlers matters!  The first handler to return True stops the chain. So given the above example,
if it's Tuesday, ``is_it_tuesday`` will return True and ``is_it_friday`` will not run.

You can override this behavior by raising the StopCheckingFeatureFlags exception in your custom handler.

    def run_only_on_tuesdays(feature):
      if date.today().weekday() == 2:
        return True
      else:
        raise StopCheckingFeatureFlags

If it isn't Tuesday, this will cause the chain to return False and any other handlers won't run.