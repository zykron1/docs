Tutorial 1
==========
In this tutorial we will create a basic project and add an index page. To create a project we need to use the
following command:

.. code-block:: bash

	python3 -m libmercury init

You will then be prompted with some information about the name of the project, as well as your python interpreter.

.. code-block:: bash

	Mercury v0.72 project initializer
	Name of directory to use: tutorial
	Please provide your python interpreter(python, python3, etc): python3
	
Then cd into the directory like so:

.. code-block:: bash

	cd tutorial

Once we have cd'd into the directory, we can see the project structure.

.. code-block:: 

	├── app.py
	├── map.json
	└── src
		├── cargo
		│   ├── connection.py
		│   ├── dev.db
		│   └── migrations
		├── controllers
		├── security
		├── static
		├── templates
		└── validators

At first this may feel overwhelming, but in practice it is quite simple. Inside the app.py file we have our WSGI
application. We can then run the app like so:

.. code-block:: bash

   python3 -m libmercury run

If this works, you should see the following output:

.. code-block:: bash

	WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
	* Running on http://localhost:8000
	Press CTRL+C to quit

Currently if we go to `localhost:8000 <http://localhost:8000>`_ we will see a 404 error. This is because currently,
there are no routes set up. To create a route we can create a controller like so:

.. code-block:: bash

	python3 -m libmercury create controller index

Once this we can see that it creted a file in the src/controllers directory called indexController.py:

.. code-block:: python3
	
	from libmercury import GETRoute, Request, Response
	class indexController:
		@staticmethod
		@GETRoute("/example")
		def example(request: Request) -> Response:
			response = Response("<h1>Example Page</h1>")
			response.headers['Content-Type'] = 'text/html'
			return response

We can modify this file so that it returns an html page. But hard coding an entire html file is not best practice so
we can instead create a template file in src/templates called index.html.

.. code-block:: html

	<html>
	<body>
		<h1>Hello World!</h1>
	</body>
	</html>

Then we can integerate this template into our controller like so:

.. code-block:: python3
	
	from libmercury import GETRoute, Request, Response, use_template
	class indexController:
		@staticmethod
		@GETRoute("/index")
		def index(request: Request) -> Response:
			return use_template("index.html") 


Now if we start our server and visit localhost:8000 we should see the html page:

And voila, you successfully created your first Mercury project!

Next tutorial: `Tutorial 2 <tutorial2.html>`_
