Tutorial 4
==========
This tutorial will be based of the `previous tutorial <tutorial3.html>`_. Please complete it first.

Doing stuff that that actually matters
--------------------------------------
So as for the past few tutorials, we have done nothing that actually does stuff. So in this tutorial, we will cover 
the JWT system and use it to create an authentication flow. To start, lets create a JWT generator with keypairs, to do
so, we need to run the following command:

.. code-block:: bash

	python3 -m libmercury create jwt Main

When we run this, it will prompt us to choose betwen HMAC or RSA. We will choose RSA.

.. code-block:: bash

	What kind of key do you want to generate? (HMAC/RSA): RSA
	[KEYGEN] RSA private key saved to src/.vault/MainPublic_key.pem
	[KEYGEN] RSA public key saved to src/.vault/MainPrivate_key.pem
	[CODEGEN] Successfully created src/cargo/MainJwt.py

Once we do this, we can use the useAutherization decorator to protect our endpoints. So lets create all our templates.
To start, lets create a signin template by creating a file in src/templates/signin.html:

.. code-block:: html

	<html>
	<body>
	<form action="/api/signin" method="post">
		<input type="text" name="username">
		<input type="password" name="password">
		<input type="submit">
	</form>
	</body>
	</html>

Then lets create a signup template by creating a file in src/templates/signup.html:

.. code-block:: html

	<html>
	<body>
	<form action="/api/signup" method="post">
		<input type="text" name="username">
		<input type="password" name="password">
		<input type="submit">
	</form>
	</body>
	</html>

Then lets create a protected endpoint that tells us some information about who is signed in. Lets create a file in 
src/templates/protected.html:

.. code-block:: html

	<html>
	<body>
		<h1>Hello {{username}}</h1>
	</body>
	</html>

Now lets rig our views up to a controller, we will name this controller "PagesController". To do so run the following 
command:

.. code-block:: bash

	python3 -m libmercury controller Pages

Then lets program our controller like so:

.. code-block:: python3

	from libmercury import GETRoute, Request, Response, use_template, useAutherization, redirect
	from src.security.MainJwt import MainJwt
	class PagesController:
		@staticmethod
		@GETRoute("/signin")
		def signin(request: Request) -> Response:
			return use_template("signin.html")
	
		@staticmethod
		@GETRoute("/signup")
		def signup(request: Request) -> Response:
			return use_template("signin.html")
	
		@staticmethod
		@GETRoute("/protected")
		@useAutherization(MainJwt, cookie="token", error=lambda: redirect("/signin"))
		def protected(request: Request) -> Response:
			return use_template("protected.html")

.. note:: 
	
	You may get an error from your IDE along the lines of it failing to find src.security.MainJwt. This is not a bug
	and is completely expected.

Writing Controllers
-------------------
Now currently, there is a big issue in the fact that anyone even if logged in can visit the /signin and /signup
pages. To fix this, we can add a check to see if our user is logged in already, we can do this like so:

.. code-block:: python3

	from libmercury import GETRoute, Request, Response, use_template, useAutherization
	from libmercury.security.jwt import JWT
	from src.security.MainJwt import MainJwt
	class PagesController:
		@staticmethod
		@GETRoute("/signin")
		def signin(request: Request) -> Response:
			if mainJwt._verify(request.cookies.get("token")):
				return redirect("/protected")
			return use_template("signin.html")
	
		@staticmethod
		@GETRoute("/signup")
		def signup(request: Request) -> Response:
			if mainJwt._verify(request.cookies.get("token")):
				return redirect("/protected")
			return use_template("signup.html")
	
		@staticmethod
		@GETRoute("/dashboard")
		@useAutherization(MainJwt, cookie="token", error=lambda: redirect("/signin"))
		def protected(request: Request) -> Response:
			return use_template("protected.html", username=JWT(request.cookies["token"]).payload["username"])

Now we can implement a login POST by opening the already created src/controllers/AuthenticationFlowController.py:

.. code-block:: python3

	from src.security.MainJwt import MainJwt

	class AuthFlowController:
		@staticmethod
		@POSTRoute("/api/signin")
		@useValidator(SigninValidator)
		def signin(request: Request) -> Response:
			if MainJwt._verify(request.cookies.get("token")):
				return redirect("/dashboard")
	
			#Check to see if the account exists
			username = request.form["username"]
			if not exists(user, username=username):
				return redirect("/signin?error=User: {username} doesn't exist")
	
			#Check to see if the password is correct
			password = request.form["password"]
			if not query(user, username=username).first().password == password:
				return redirect("/signin?error=Incorrect password")
	
			response = redirect("/dashboard")
			response.set_cookie("token", MainJwt._makeJwt({"username": username, "exp": expires_in(3600)}))
			return response 
	
		@staticmethod
		@POSTRoute("/api/signup")
		@useValidator(SignupValidator, error = lambda: redirect("/signup"))
		def signup(request: Request) -> Response:
			if MainJwt._verify(request.cookies.get("token")):
				return redirect("/dashboard")
	
			#Check to see if an account exists
			username = request.form["username"]
			if exists(user, username=username):
				return redirect(f"/signup?error=Account: {username} already exists") 
	
			#Create the account
			password = request.form["password"]
			new_user = user(username=username, password=password)
			Connection.Session.add_all([new_user,])
			Connection.Session.commit()
	
			#Redirect to the login page
			return redirect("/signin")

Testing it out
--------------
If you run the server and then go to `this page <http://localhost:8000/signin>`_ 

Next tutorial: `Tutorial 5 <tutorial5.html>`_
