Tutorial 5
==========
This tutorial will be based of the `previous tutorial <tutorial4.html>`_. Please complete it first.

Autogeneration
--------------
In the last tutorial we manualy created a website with a login and signup system. But what if I told you, we could do
all of that with just the CLI? To do so, lets create a new project from scratch:

.. code-block:: bash

	python3 -m libmercury init

Then once we cd into our project folder we can use the CLI to create a login system:

.. code-block:: 

	python3 -m libmercury generate control:login
	[GENERATOR] Starting generation task
	What should be the name of the controller: Login
	What should be the route of the controller: /api/login
	What should be the success redirect of the controller: /dashboard
	What should be the name of the validator: Login
	What is the name of the model to sign into(leave blank to make a new one):
	What should be the table's name: user
	What should the username field be called: username
	What should the password field be called: password
	Is the password hashed? (y/n): y
	Is there an email field? (y/n): y
	What should the email field be called: email
	[CODEGEN] Successfully created src/cargo/userModel.py
	What should the salt be called: salt
	[GENERATOR] Successfully created model
	What is the name of the model to sign into(leave blank to make a new one): user
	What is the name of the model's password field: password
	What is the name of the model's username field: username
	Is the password hashed? (y/n): y
	What is the name of the validator function(check_password if you used this for the model): check_password
	What should be the name of the JWT: Main
	What kind of key do you want to generate? (HMAC/RSA): rsa
	[KEYGEN] RSA private key saved to src/.vault/MainPublic_key.pem
	[KEYGEN] RSA public key saved to src/.vault/MainPrivate_key.pem
	[CODEGEN] Successfully created src/cargo/MainJwt.py
	[CODEGEN] Successfully created src/controllers/LoginController.py
	[CODEGEN] Successfully created src/validators/LoginValidator.py
	[GENERATOR] Successfully created login controller

And just like that, it created a user model, as well as a table. All we have to do now, is just set up the migration
in our database, we can do so with the CLI:

.. code-block:: bash

	python3 -m libmercury create migration first_migration

Then we can program the migration to create the table:

.. code-block:: python3

	from libmercury.db import Integer, MigrationWrapper, String, Column
	
	_version = '1'
	_prev_version = None
	
	def upgrade(url):
		wrapper = MigrationWrapper(url)
		wrapper.create_table("user", [
			Column("id", Integer, primary_key=True),
		Column("username", String(20)),
		Column("password", String(50)),
		Column("salt", String, nullable=False),
		Column("email", String(318)),
		])
	
	def downgrade(url):
		wrapper = MigrationWrapper(url)

Now let's use our migration with the CLI:

.. code-block:: bash

	python3 -m libmercury migrate

With our model created, we can now easily create a register for our model with the CLI:

.. code-block:: 

	python3 -m libmercury generate control:register

	[GENERATOR] Starting generation task
	What should be the name of the controller: Register
	What should be the route of the controller: /api/register
	What should be the success redirect of the controller: /login 
	What should be the name of the validator: Register
	What is the name of the model to register for(leave blank to make a new one): user
	Provide the name of your JWT to make sure that users are not signed in, leave blank to make a new one: Main
	What is the name of the model's username field: username
	Is the password hashed? (y/n): y 
	What is the name of the model's password field: password
	What is the name of the model's password setter field: set_password
	Where should the controller redirect if the user is already signed in: /dashboard
	What is the name of the model's primary key: id
	[CODEGEN] Successfully created src/validators/RegisterValidator.py
	[CODEGEN] Successfully created src/controllers/RegisterController.py

Making it pretty
----------------

Once we do this, we can now set up all our pages. So lets create all our templates.
To start, lets create a signin template by creating a file in src/templates/login.html:

.. code-block:: html

	<html>
	<body>
	<form action="/api/login" method="post">
		<input type="text" name="username">
		<input type="password" name="password">
		<input type="submit">
	</form>
	</body>
	</html>

Then lets create a signup template by creating a file in src/templates/register.html:

.. code-block:: html

	<html>
	<body>
	<form action="/api/register" method="post">
		<input type="text" name="username">
		<input type="password" name="password">
		<input name="email">
		<input type="submit">
	</form>
	</body>
	</html>

Then lets create a protected endpoint that tells us some information about who is signed in. Lets create a file in 
src/templates/dashboard.html:

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
	from libmercury.security.jwt import JWT
	from src.security.MainJwt import MainJwt
	class PagesController:
		@staticmethod
		@GETRoute("/loign")
		def signin(request: Request) -> Response:
			return use_template("login.html")
	
		@staticmethod
		@GETRoute("/register")
		def signup(request: Request) -> Response:
			return use_template("register.html")
	
		@staticmethod
		@GETRoute("/dashboard")
		@useAutherization(MainJwt, cookie="token", error=lambda: redirect("/signin"))
		def protected(request: Request) -> Response:
			return use_template("dashboard.html", username=JWT(request.cookies["token"]).payload["username"])


And just like that, if we visit `our page <localhost:8000/register>`_ with the server running, we should be able to 
use our website.

Next tutorial: `Tutorial 6 <tutorial6.html>`_
