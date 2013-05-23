Flask-GoogleAuth
================
This is a partial port of torando.auth to be used with Flask.

Written by Alexander Saltanov, inspired by Kenneth Reitz.


Changes in Runscope fork
------------------------
1. Set the cookie name via a parameter. (Before it was hard-coded to "openid".)
   This lets us invalidate existing cookies by changing cookie_name. We'll need
   to do that whenever we make changes to the g.user structure, or when we
   remove an authorized user from Google Apps. (Once the auth cookie is set,
   the extension never re-confirms with Google. If a user leaves the company,
   this is currently the only way to invalidate their existing sessions.)
2. Require auth for all endpoints in an app, by adding required=True to the
   auth object constructor.
3. Adds /auth/ endpoint, which can be used with the nginx module
   http_auth_request to authenticate other endpoints outside Flask. /auth/
   returns a 200 OK response if the user's session can be authenticated, and
   401 Unauthorized if it cannot.
4. Adds token-based authentication for API access, by way of an Authentication
   header. If session-based auth fails, we check for an Authorization header
   matching this pattern: "Authorization: token <TOKEN-VALUE>". A list of valid
   tokens are currently hard-coded in the extension, but this will eventually
   be modified to pull from an internal Runscope service.


Feature requests for Runscope fork
----------------------------------
1. Support token-based auth for scripts that interact with APIs. We'll need to
   validate the tokens with a database (maybe identity?) or hard-code the valid
   tokens into this script.
2. Load a list of valid users from a database. This would let us limit certain
   services to authorized employees, or remove a user who's left the company.


Usage
-----
Example usage for Google Federated Login. Greate for internal apps.

Routes ``/login/`` and ``/logout/`` will be provided automagically.

Require an account from a given Google Apps domain for your Flask apps::

    from flask import Flask, g
    from flask_googleauth import GoogleFederated

    # Setup Flask
    app = Flask(__name__)
    app.secret_key = "random secret key"

    # Setup Google Federated Auth
    auth = GoogleFederated("mokote.com", app)

    @app.route("/")
    @auth.required
    def secret():
        # Once user is authenticated, his name and email are accessible as
        # g.user.name and g.user.email.
        return "You have rights to be here, %s (%s)" % (g.user.name, g.user.email)

    app.run()

If you want to authenticate your users with general Google OpenID you should import and use ``GoogleAuth`` instead of ``GoogleFederated``::

    auth = GoogleAuth(app)


Install
-------
To install Flask-GoogleAuth::

    pip install flask-googleauth


Prerequisites
-------------
Be sure that your Google Apps domain is enabled to be an OpenID provider under "Advanced tools" → "Federated Login using OpenID".
