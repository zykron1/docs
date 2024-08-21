Tutorial 5
==========
This tutorial builds on the `previous tutorial <tutorial4.html>`_. Please complete it before proceeding.

Autogeneration
--------------
In the previous tutorial, we manually created a website with a login and signup system. But what if I told you that we could do all of this with just the CLI? Let's create a new project from scratch:

.. code-block:: bash

    python3 -m libmercury init

After navigating to your project folder, you can use the CLI to generate a login system:

.. code-block:: bash

    python3 -m libmercury generate control:login

You will then be prompted with several questions to configure your login system:

.. code-block:: bash

    [GENERATOR] Starting generation task
    What should be the name of the controller: Login
    What should be the route of the controller: /api/login
    What should be the success redirect of the controller: /dashboard
    What should be the name of the validator: Login
    What is the name of the model to sign into (leave blank to create a new one):
    What should the table's name be: user
    What should the username field be called: username
    What should the password field be called: password
    Is the password hashed? (y/n): y
    Is there an email field? (y/n): y
    What should the email field be called: email
    [CODEGEN] Successfully created src/cargo/userModel.py
    What should the salt be called: salt
    [GENERATOR] Successfully created model
    What is the name of the model to sign into (leave blank to create a new one): user
    What is the name of the model's password field: password
    What is the name of the model's username field: username
    Is the password hashed? (y/n): y
    What is the name of the validator function (check_password if you used this for the model): check_password
    What should be the name of the JWT: Main
    What kind of key do you want to generate? (HMAC/RSA): rsa
    [KEYGEN] RSA private key saved to src/.vault/MainPrivate_key.pem
    [KEYGEN] RSA public key saved to src/.vault/MainPublic_key.pem
    [CODEGEN] Successfully created src/cargo/MainJwt.py
    [CODEGEN] Successfully created src/controllers/LoginController.py
    [CODEGEN] Successfully created src/validators/LoginValidator.py
    [GENERATOR] Successfully created login controller

With just a few commands, a user model and corresponding table are created. Next, we need to set up the migration in our database, which can be done using the CLI:

.. code-block:: bash

    python3 -m libmercury create migration first_migration

Now, let's program the migration to create the table:

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
        wrapper.delete_table("user")

To apply the migration, use the following CLI command:

.. code-block:: bash

    python3 -m libmercury migrate

Now that our model is created, we can easily generate a registration system using the CLI:

.. code-block:: bash

    python3 -m libmercury generate control:register

You will be prompted with configuration questions for the registration system:

.. code-block:: bash

    [GENERATOR] Starting generation task
    What should be the name of the controller: Register
    What should be the route of the controller: /api/register
    What should be the success redirect of the controller: /login 
    What should be the name of the validator: Register
    What is the name of the model to register for (leave blank to create a new one): user
    Provide the name of your JWT to ensure that users are not signed in (leave blank to create a new one): Main
    What is the name of the model's username field: username
    Is the password hashed? (y/n): y 
    What is the name of the model's password field: password
    What is the name of the model's password setter field: set_password
    Where should the controller redirect if the user is already signed in: /dashboard
    What is the name of the model's primary key: id
    [CODEGEN] Successfully created src/validators/RegisterValidator.py
    [CODEGEN] Successfully created src/controllers/RegisterController.py

Making it Pretty
----------------
With the back-end in place, we can now set up all our pages. Let's start by creating a sign-in template in `src/templates/login.html`:

.. code-block:: html

    <html>
    <body>
    {% if error %}
        <div class="error">{{ error }}</div>
    {% endif %}
    <form action="/api/login" method="post">
        <input type="text" name="username">
        <input type="password" name="password">
        <input type="submit">
    </form>
    </body>
    </html>

Next, create a sign-up template in `src/templates/register.html`:

.. code-block:: html

    <html>
    <body>
        {% if error %}
            <div class="error">{{ error }}</div>
        {% endif %}
    <form action="/api/register" method="post">
        <input type="text" name="username">
        <input type="password" name="password">
        <input name="email">
        <input type="submit">
    </form>
    </body>
    </html>

Finally, create a protected endpoint that displays information about the signed-in user in `src/templates/dashboard.html`:

.. code-block:: html

    <html>
    <body>
        <h1>Hello {{username}}</h1>
    </body>
    </html>

Now let's connect these views to a controller named `PagesController`. To create this controller, run the following command:

.. code-block:: bash

    python3 -m libmercury controller Pages

Then, add the following code to your controller:

.. code-block:: python3

    from libmercury import GETRoute, Request, Response, use_template, useAuthorization, redirect
    from libmercury.security.jwt import JWT
    from src.security.MainJwt import MainJwt
    
    class PagesController:
        @staticmethod
        @GETRoute("/login")
        def signin(request: Request) -> Response:
            return use_template("login.html", error=request.args.get("error"))
    
        @staticmethod
        @GETRoute("/register")
        def signup(request: Request) -> Response:
            return use_template("register.html", error=request.args.get("error"))
    
        @staticmethod
        @GETRoute("/dashboard")
        @useAuthorization(MainJwt, cookie="token", error=lambda: redirect("/login"))
        def protected(request: Request) -> Response:
            return use_template("dashboard.html", username=JWT(request.cookies["token"]).payload["username"])

With this setup, if you visit `your page <localhost:8000/register>`_ while the server is running, you should be able to use your website.

Next tutorial: `Tutorial 6 <tutorial6.html>`_
