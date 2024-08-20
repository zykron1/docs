Tutorial 1: Getting Started with Your Mercury Project
=====================================================

In this tutorial, we'll guide you through the process of creating a basic project and adding an index page. To begin, initialize your project with the following command:

.. code-block:: bash

	python3 -m libmercury init

You will be prompted to provide the name of the project and specify your Python interpreter.

.. code-block:: bash

	Mercury v0.72 project initializer
	Name of directory to use: tutorial
	Please provide your Python interpreter (python, python3, etc.): python3

After setting up your project, navigate to the newly created directory:

.. code-block:: bash

	cd tutorial

Within this directory, you'll find the following project structure:

.. code-block:: none

	├── app.py
	├── map.json
	└── src
		├── cargo
		│	├── connection.py
		│	├── dev.db
		│	└── migrations
		├── controllers
		├── security
		├── static
		├── templates
		└── validators

While this structure may seem complex at first, it’s organized to streamline your development process. The ``app.py`` file contains your WSGI application, which you can run using:

.. code-block:: bash

	python3 -m libmercury run

If successful, you'll see the following output:

.. code-block:: bash

	WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
	* Running on http://localhost:8000
	Press CTRL+C to quit

At this stage, visiting `http://localhost:8000` will result in a 404 error because no routes are currently configured. To address this, let’s create a new controller:

.. code-block:: bash

	python3 -m libmercury create controller index

This command generates a file named ``indexController.py`` in the ``src/controllers`` directory:

.. code-block:: python

	from libmercury import GETRoute, Request, Response

	class indexController:
		@staticmethod
		@GETRoute("/example")
		def example(request: Request) -> Response:
			response = Response("<h1>Example Page</h1>")
			response.headers['Content-Type'] = 'text/html'
			return response

While this controller returns a simple HTML response, hardcoding HTML is not best practice. Instead, let's create a template file named ``index.html`` in the ``src/templates`` directory:

.. code-block:: html

	<html>
	<body>
		<h1>Hello World!</h1>
	</body>
	</html>

Next, integrate this template into your controller:

.. code-block:: python

	from libmercury import GETRoute, Request, Response, use_template

	class indexController:
		@staticmethod
		@GETRoute("/index")
		def index(request: Request) -> Response:
			return use_template("index.html")

Now, restart your server and visit `this page <http://localhost:8000>`_ to see your HTML page in action.

Congratulations! You have successfully created your first Mercury project.

Next tutorial: `Tutorial 2 <Tutorial2.html>`_ 
